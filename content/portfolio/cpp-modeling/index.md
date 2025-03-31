+++
authors = ["Lucien Gheerbrant"]
title = "C++ modelling of a custom RISC-V core"
date = 2024-08-10
[markdown]
numbersections = true
[taxonomies]
tags = ["C++", "Python", "Computer Architecture", "RTL", "RISC-V"]
[extra]
featured = true
toc = true
katex = true
+++

{% alert(note=true) %}
I have truncated a big part of my work. A lot of what I worked on was proprietary design owned by <a class='external' href='https://www.prophesee.ai/'>PROPHESEE</a>. I only kept the part of the design that was open source and already public here, to avoid any problem.
{% end %}


## Introduction

During an internship at <a class='external' href='https://www.prophesee.ai/'>PROPHESEE</a>, I had the opportunity to work on a fascinating project involving the simulation of a RISC-V based system using the <a class="external" href="https://www.gem5.org/">gem5</a> simulator. This article focuses on the C++ modelling aspects of the project, highlighting the challenges and achievements in creating a simulation model for an event-based camera system.

The primary goal of my internship was to simulate <a class="external" href="https://syntacore.com/products/scr1">the SCR1 microcontroller</a> for an event-based camera using gem5. This involved creating detailed models of the system's components, including the SCR1 processor, memory, and peripherals like timers and a MailBox to use with FreeRTOS.

## The SCR1 Microcontroller

The SCR1 by Syntacore is an <a class="external" href="https://github.com/syntacore/scr1">open-source</a> microcontroller. It is built on the RV32I architecture, a subset of the 32-bit RISC-V architecture. It comprises 47 instructions, supporting arithmetic, logical, load, and store operations. The SCR1 is designed to meet the needs of embedded systems and Internet of Things (IoT) applications, characterised by its energy efficiency and minimalistic design.

The SCR1, as depicted below, employs a pipeline architecture with stages ranging from two to four. It features an Integer Processing Unit (IPU) for instruction decoding and execution, along with a memory interface. The memory interfaces can be configured to use either the Advanced eXtensible Interface (AXI) or Advanced High-performance Bus (AHB) bus protocols in the RTL files. Additionally, the core includes an optional cache controller to optimise data access.

<figure>
    {{
        image(
            url_min='https://syntacore.com/media/scr1/scr1-isometric.svg',
            url='https://syntacore.com/media/scr1/scr1-site-.png',
            transparent=true
        )
    }}
    <figcaption>
        Schematics of the SCR1 (Click for more details)
    </figcaption>
</figure>

The processor also includes an Integrated Programmable Interrupt Controller (IPIC), which provides a rapid response to events from peripherals external to the core. Internally, the SCR1 has sources of interrupts, such as the timer, which operates according to RISC-V standards and is directly connected to the Interrupt Controller (IT CTRL).

The IPIC differs from the IT CTRL in that it manages all interrupt codes, both internal and external, but cannot differentiate between various external sources. The RISC-V architecture includes only one code for external interrupts, necessitating the IPIC to handle multiple external interrupt routines and prioritise them accordingly.

Lastly, the SCR1 includes a dedicated debug block and optionally uses a Tightly Coupled Memory (TCM) architecture. TCM integrates the Data Memory (DMEM) and Instruction Memory (IMEM) into a single unit close to the processor, reducing latency in accessing the main memory in systems using SRAM, effectively mimicking cache functionality.

## gem5

### the gem5 simulator

The gem5 simulator is an <a class="external" href="https://www.gem5.org/">open-source</a> project, maintained and utilised by numerous individuals and entities, including major tech companies like Google and ARM, as well as prominent universities worldwide. The project is continuously evolving, with its feature set expanding over the years. The gem5 simulator enables the modelling via C++ and interaction of computer system components such as processors, memory, and buses. Each component can be represented with varying levels of detail, offering a wide range of architectural choices.

### Using gem5

To understand how gem5 is used, let's simplify the process of creating a model within this simulator.

The gem5 simulator primarily uses C++, with some Python for configuration. It is object-oriented, with the C++ code consisting of a series of classes. These classes represent both abstract objects, such as the system clock frequency or instruction set architecture, and concrete objects like DDR3 RAM or a processor. These objects are known as SimObjects. The Python code acts as configuration files, linking and connecting the various C++ objects and initiating the simulation.

The first step in gem5 is to create the "system," an object that defines the simulation environment:

```python
system = System()
```

Next, you can add other objects as attributes to the system. For example, let's add a clock frequency:

```python
system.clk_domain = SrcClockDomain()
system.clk_domain.clock = '50MHz'
```

This clock frequency will be used by other simulated objects. A processor model named `RiscvMinorCpu`, which uses the RISC-V instruction set, is an example:

```python
system.cpu = RiscvMinorCPU()
```

Now, let's connect a memory bus. The gem5 project modifies the assignment operator `=` to connect ports:

```python
system.membus = SystemXBar()
system.cpu.icache_port = system.membus.cpu_side_ports
system.cpu.dcache_port = system.membus.cpu_side_ports
```

This model is illustrated below:

<figure>
    <center>
        <object type="image/svg+xml" data="img/demo.svg" width="50%">
            {{
                image(
                    url='img/demo.svg',
                    transparent=true,
                    no_hover=true
                )
            }}
        </object>
    </center>
    <figcaption>
        Graph of the processor-bus connection written in Python above
    </figcaption>
</figure>

## Modelling!

The gem5 simulator is designed for more generic computer systems. The processors it models often support multicore architectures, MMU, etc. It was necessary to modify the code to make it as close as possible to the MCU and remove complex functionalities from the "ready-to-use" models written in the gem5 project.

The core model is derived from the "Minor" CPU model. Minor is a processor model with configurable execution behaviour. For example, you can modify the execution delay of an integer-processing instruction or memory write.

To better model SCR1, several changes were applied, such as using the RV32I instruction set instead of RV64. Several processing units not present in the SCR1 but modelled in gem5's Minor processor model were removed. These units handled floating-point operations, vectors, etc. The figure bellow shows the minimised instruction set of the SCR1 compared to the standard RV32 and RV64:

<figure>
    {{
        image(
            url='img/SCR1Units.svg',
            transparent=true,
            no_hover=true
        )
    }}
    <figcaption>
        SCR1 instruction set
    </figcaption>
</figure>


The simulator allows configuring numerous parameters in the model to achieve performance close to the real system. For example, the SCR1 has a latency of 34 cycles during the execution of an integer division. To reflect this in the model, it is sufficient to configure an attribute of a given Python class in the gem5 library. This class represents the integer division processing unit:

```python
class SCR1IntDivFU(MinorDefaultIntDivFU):
    opLat = 34
```

`SCR1IntDivFU` will be reused in the core class definition, allowing the simulator to know how long a division should take.

### IPIC and the CSR

To model these modules, it was necessary to create `SimObjects`, some of which were already implemented in the gem5 library. The technical specifications of the SCR1 describe registers dedicated to the IPIC in the range `[0xBF0:0xBF7]`.

This range `[0xBF0:0xBF7]` does not correspond to a usual memory range, but to a zone within the Control and Status Registers (CSR). The CSRs are special registers in the RISC-V architecture. They are defined in the Zicsr extension. These are internal registers that control interrupts, exceptions, and other crucial aspects of the processor's state and operation. These are already modelled in the gem5 simulator, but in a standard way as described in the RISC-V manual.

<figcaption>
CSR non-standards implemented in the modified RISC-V architecture of the SCR1
</figcaption>

| Address        | Name           |
| -------------- | -------------- |
| `0x7E0`        | `MCOUNTEN`     |
| `0xBF0..0xBF7` | IPIC Registers |


However, if the "standard" CSRs were already modeled, the SCR1 processor contains its own registers dedicated to the IPIC. This is not accounted for by the gem5 models because it is a non-standard implementation, specific to the SCR1. Fortunately, thanks to the open-source philosophy of gem5, finding the commit that adds the part of the code modeling the CSRs was quite simple. A class `SCR1ISA` was created to model this modification of the RISC-V instruction set. This class inherits from the `RiscvISA` class provided by gem5 and adds the logic related to the IPIC. These registers contain the parameters of the external interrupts, such as activation on the rising or falling edge, or the activation of the interrupt itself.

<details>

<summary>Header file of the custom RISC-V ISA</summary>


```cpp
#ifndef DEV_SCR1_SCR1_ISA_HH
#define DEV_SCR1_SCR1_ISA_HH

#include "arch/riscv/interrupts.hh"
#include "arch/riscv/isa.hh"
#include "cpu/base.hh"
#include "debug/RiscvMisc.hh"
#include "debug/SCR1ISA.hh"
#include "params/SCR1ISA.hh"
#include "sim/system.hh"

namespace gem5 {
namespace RiscvISA {

enum NonStandardMiscRegIndex
{
  // SCR1 non-standard registers

  MISCREG_COUNTEN = NUM_MISCREGS,

  MISCREG_IPIC_CISV,
  MISCREG_IPIC_CICSR,
  MISCREG_IPIC_IPR,
  MISCREG_IPIC_ISVR,
  MISCREG_IPIC_EOI,
  MISCREG_IPIC_SOI,
  MISCREG_IPIC_IDX,
  MISCREG_IPIC_ICSR,

  TOTAL_NUM_MISCREG,

  NONSTANDARD_NUM_MISCREG = TOTAL_NUM_MISCREG - NUM_MISCREGS,
};

enum IpicParams
{
  IRQ_VECT_NUM = 16,
  IRQ_VECT_WIDTH = ceilLog2(IRQ_VECT_NUM + 1),
  IRQ_LINES_NUM = IRQ_VECT_NUM,
  IRQ_LINES_WIDTH = ceilLog2(IRQ_LINES_NUM),
  IRQ_VOID_VECT_NUM = IRQ_VECT_NUM // According to SCR1 EAS
};

enum NonStandardIndex
{
  // SCR1 non-standard registers

  CSR_MCOUNTEN = 0x7E0,

  CSR_IPIC_CISV = 0xBF0 + 0x00,
  CSR_IPIC_CICSR = 0xBF0 + 0x01,
  CSR_IPIC_IPR = 0xBF0 + 0x02,
  CSR_IPIC_ISVR = 0xBF0 + 0x03,
  CSR_IPIC_EOI = 0xBF0 + 0x04,
  CSR_IPIC_SOI = 0xBF0 + 0x05,
  CSR_IPIC_IDX = 0xBF0 + 0x06,
  CSR_IPIC_ICSR = 0xBF0 + 0x07,
};
class SCR1ISA : public ISA
{
private:
  std::unordered_map<int, CSRMetadata> SCR1CSRData = CSRData;

  std::unordered_map<int, std::string> NonStandardMiscRegNames = {
      // SCR1 non-standard registers
      {MISCREG_COUNTEN, "COUNTEN"},

      {MISCREG_IPIC_CISV, "IPIC_CISV"}, {MISCREG_IPIC_CICSR, "IPIC_CICSR"},
      {MISCREG_IPIC_IPR, "IPIC_IPR"},   {MISCREG_IPIC_ISVR, "IPIC_ISVR"},
      {MISCREG_IPIC_EOI, "IPIC_EOI"},   {MISCREG_IPIC_SOI, "IPIC_SOI"},
      {MISCREG_IPIC_IDX, "IPIC_IDX"},   {MISCREG_IPIC_ICSR, "IPIC_ICSR"}};

  std::unordered_map<int, CSRMetadata> NonStandardCSRData = {
      {CSR_MCOUNTEN,
       {"mcounten", MISCREG_COUNTEN, rvTypeFlags(RV32), isaExtsFlags()}},
      {CSR_IPIC_CISV,
       {"ipic_cisv", MISCREG_IPIC_CISV, rvTypeFlags(RV32), isaExtsFlags()}},
      {CSR_IPIC_CICSR,
       {"ipic_cicsr", MISCREG_IPIC_CICSR, rvTypeFlags(RV32), isaExtsFlags()}},
      {CSR_IPIC_IPR,
       {"ipic_ipr", MISCREG_IPIC_IPR, rvTypeFlags(RV32), isaExtsFlags()}},
      {CSR_IPIC_ISVR,
       {"ipic_isvr", MISCREG_IPIC_ISVR, rvTypeFlags(RV32), isaExtsFlags()}},
      {CSR_IPIC_EOI,
       {"ipic_eoi", MISCREG_IPIC_EOI, rvTypeFlags(RV32), isaExtsFlags()}},
      {CSR_IPIC_SOI,
       {"ipic_soi", MISCREG_IPIC_SOI, rvTypeFlags(RV32), isaExtsFlags()}},
      {CSR_IPIC_IDX,
       {"ipic_idx", MISCREG_IPIC_IDX, rvTypeFlags(RV32), isaExtsFlags()}},
      {CSR_IPIC_ICSR,
       {"ipic_icsr", MISCREG_IPIC_ICSR, rvTypeFlags(RV32), isaExtsFlags()}}};

public:
  PARAMS(SCR1ISA);
  SCR1ISA(const Params &params);

  RegVal readMiscReg(RegIndex idx) override;
  RegVal readMiscRegNoEffect(RegIndex idx) const override;
  void setMiscReg(RegIndex idx, RegVal val) override;
  void setMiscRegNoEffect(RegIndex idx, RegVal val) override;

  const std::unordered_map<int, CSRMetadata> &getCSRDataMap() const override {
    return SCR1CSRData;
  }

  void clearIp(int id);

  typedef struct icsrMetadata
  {
    bool ie;
    bool im;
    bool inv;
  } icsrMetadata;

  icsrMetadata icsrData[IRQ_VECT_NUM];

private:
  System *system;
};
} // namespace RiscvISA
} // namespace gem5

#endif /* DEV_SCR1_SCR1_ISA_HH */
```

</details>

<details>

<summary>C++ definitions of the custom RISC-V ISA</summary>


```cpp
#include "dev/scr1/scr1_isa.hh"

namespace gem5 {
namespace RiscvISA {

SCR1ISA::SCR1ISA(const Params &params) : ISA(params) {
  SCR1CSRData.insert(NonStandardCSRData.begin(), NonStandardCSRData.end());

  miscRegFile.resize(TOTAL_NUM_MISCREG);
  clear();
}

void SCR1ISA::setMiscReg(RegIndex idx, RegVal val) {
  if (idx >= MISCREG_COUNTEN && idx <= MISCREG_IPIC_ICSR) {
    switch (idx) {
    case MISCREG_IPIC_CISV:
    case MISCREG_IPIC_ISVR: {
    } break;

    case MISCREG_IPIC_CICSR: {
      RegVal cisvNumber = bits(readMiscRegNoEffect(MISCREG_IPIC_CISV), 3, 0);
      if (cisvNumber < IRQ_VOID_VECT_NUM) {
        if (bits(val, 0)) {
          clearIp(cisvNumber);
        }
        icsrData[cisvNumber].ie = bits(val, 1);
      }
    } break;

    case MISCREG_IPIC_IPR: {
      for (RegVal i = 0; i < IRQ_VECT_NUM; i++) {
        if (bits(val, i)) {
          clearIp(i);
        }
      }
    } break;

    case MISCREG_IPIC_EOI: {
      // End Of Interrupt: We need to find the next highest priority interrupt
      // in service to run.
      RegVal id = 0;
      RegVal isvr = readMiscRegNoEffect(MISCREG_IPIC_ISVR);

      auto tc = system->threads[0];
      tc->getCpuPtr()->clearInterrupt(tc->threadId(),
                                      ExceptionCode::INT_EXT_MACHINE, 0);

      while (id < IRQ_VECT_NUM && !bits(isvr, id)) {
        id++;
      }
      setMiscRegNoEffect(MISCREG_IPIC_CISV, id);
      if (id < IRQ_VECT_NUM) {
        RegVal cicsr = 0;
        cicsr |= bits(readMiscRegNoEffect(MISCREG_IPIC_IPR), id);
        cicsr |= icsrData[id].ie << 1;
        setMiscRegNoEffect(MISCREG_IPIC_CICSR, cicsr);

        isvr &= ~(1ULL << id);
        setMiscRegNoEffect(MISCREG_IPIC_ISVR, isvr);
      } else {
        setMiscRegNoEffect(MISCREG_IPIC_CICSR, 0);
      }
    } break;

    case MISCREG_IPIC_SOI: {
      bool soiCondition = false;

      RegVal cisv = readMiscRegNoEffect(MISCREG_IPIC_CISV);
      RegVal isvr = readMiscRegNoEffect(MISCREG_IPIC_ISVR);
      RegVal ipr = readMiscRegNoEffect(MISCREG_IPIC_IPR);
      for (uint id = 0; id < IRQ_LINES_NUM; id++) {
        soiCondition |= bits(ipr, id) && icsrData[id].ie && isvr == 0;
        soiCondition |= bits(ipr, id) && icsrData[id].ie && id > cisv;
      }
      if (soiCondition) {
        auto tc = system->threads[0];
        tc->getCpuPtr()->postInterrupt(tc->threadId(),
                                       ExceptionCode::INT_EXT_MACHINE, 0);
      }
    } break;

    case MISCREG_IPIC_ICSR: {
      RegVal idx = readMiscRegNoEffect(MISCREG_IPIC_IDX);
      if (idx >= IRQ_LINES_NUM) {
        break;
      }

      if (bits(val, 0)) {
        clearIp(idx);
      }
      icsrData[idx].ie = bits(val, 1);
      icsrData[idx].im = bits(val, 2);
      icsrData[idx].inv = bits(val, 3);
      RegVal isvr = readMiscRegNoEffect(MISCREG_IPIC_ISVR);
      isvr &= ~(1ULL << idx);
      isvr |= bits(val, idx) << 4;
      setMiscRegNoEffect(MISCREG_IPIC_ISVR, isvr);
    } break;

    default:
      setMiscRegNoEffect(idx, val);
    }

  } else {
    ISA::setMiscReg(idx, val);
  }
}

void SCR1ISA::setMiscRegNoEffect(RegIndex idx, RegVal val) {
  // Illegal CSR
  panic_if(idx > TOTAL_NUM_MISCREG, "Illegal CSR index %#x\n", idx);
  auto it = NonStandardMiscRegNames.find(idx);
  std::string regName =
      (it != NonStandardMiscRegNames.end()) ? it->second : "Unknown";
  DPRINTF(RiscvMisc, "Setting MiscReg %s (%d) to %#x.\n", regName, idx, val);
  miscRegFile[idx] = val;
}

RegVal SCR1ISA::readMiscReg(RegIndex idx) {
  if (idx >= MISCREG_COUNTEN && idx <= MISCREG_IPIC_ICSR) {
    switch (idx) {
    case MISCREG_IPIC_CISV: {
      RegVal cisv = readMiscRegNoEffect(MISCREG_IPIC_CISV);
      RegVal void_vect = (RegVal)IRQ_VOID_VECT_NUM;

      return (readMiscRegNoEffect(MISCREG_IPIC_ISVR) != 0) ? cisv : void_vect;
    }

    case MISCREG_IPIC_CICSR: {
      RegVal cicsr = 0;
      RegVal cisvNumber = bits(readMiscRegNoEffect(MISCREG_IPIC_CISV), 3, 0);
      RegVal ip = bits(readMiscRegNoEffect(MISCREG_IPIC_CISV), cisvNumber);

      cicsr |= ip;
      cicsr |= icsrData[cisvNumber].ie << 1;

      return cicsr;
    }

    case MISCREG_IPIC_ICSR: {
      RegVal idx = bits(readMiscRegNoEffect(MISCREG_IPIC_IDX), 3, 0);
      RegVal icsr = 0;
      icsr |= bits(readMiscRegNoEffect(MISCREG_IPIC_IPR), idx);
      icsr |= icsrData[idx].ie << 1;
      icsr |= icsrData[idx].im << 2;
      icsr |= icsrData[idx].inv << 3;
      icsr |= bits(readMiscRegNoEffect(MISCREG_IPIC_ISVR), idx) << 4;
      icsr |= 0b11ULL << 8; // Privilege mode: hardwired to 11 (M-mode)
      icsr |= idx << 12;

      return icsr;
    }

    default:
      return readMiscRegNoEffect(idx);
    }
  } else {
    return ISA::readMiscReg(idx);
  }
}

RegVal SCR1ISA::readMiscRegNoEffect(RegIndex idx) const {
  if (idx >= MISCREG_COUNTEN && idx <= MISCREG_IPIC_ICSR) {
    // Illegal CSR
    panic_if(idx > TOTAL_NUM_MISCREG, "Illegal CSR index %#x\n", idx);
    auto it = NonStandardMiscRegNames.find(idx);
    std::string regName =
        (it != NonStandardMiscRegNames.end()) ? it->second : "Unknown";
    DPRINTF(RiscvMisc, "Reading MiscReg %s (%d): %#x.\n", regName, idx,
            miscRegFile[idx]);
    return miscRegFile[idx];
  } else {
    return ISA::readMiscRegNoEffect(idx);
  }
}

void SCR1ISA::clearIp(int id) {
  RegVal ipr = readMiscRegNoEffect(MISCREG_IPIC_IPR);
  ipr &= ~(0x1ULL << id);
  setMiscRegNoEffect(MISCREG_IPIC_IPR, ipr);
}

} // namespace RiscvISA
} // namespace gem5
```

</details>

These registers allowed solving the problem described in the specifications of part 2. The execution of the program "Hello World" is now possible with this model.

<figure>
    {{
        image(
            url='img/ipicCsr.svg',
            transparent=true,
            no_hover=true
        )
    }}
    <figcaption>
        Graph representing the IPIC - interrupt source connection
    </figcaption>
</figure>


```cpp
class SCR1IPIC : public ClockedObject
{
private:
    System *system;

    using IntSink = IntSinkPin<SCR1IPIC>;
    std::vector<IntSink *> irqLines; // Signal communication between the IPIC
                                     // and others.
    int irqLinesValue = 0;           // More explicit variable to use in logic.

public:
    PARAMS(SCR1IPIC);
    SCR1IPIC(const Params &params);

    void checkINV(bool signal, int irqLine);
    void checkIM(bool signal, int irqLine);

    void raiseInterruptPin(int id) { checkINV(1, id); };
    void lowerInterruptPin(int id) { checkINV(0, id); };

    Port &getPort(const std::string &if_name, PortID idx) override;
};
```
<figcaption>
IPIC C++ class declaration
</figcaption>

<figure>
    {{
        image(
            url='img/SCR1_core.svg',
            transparent=true,
            no_hover=true
        )
    }}
    <figcaption>
        Graph representing the SCR1 core
    </figcaption>
</figure>

### Memory and memory bus

The SCR1 does not have a cache. It only includes DMEM and IMEM, which are directly part of the SRAM. There is no additional layer, such as an MMU. However, it is necessary to implement the cache hierarchy. But the model does not actually have one. This might seem confusing, but it is a side effect of gem5, which prefers generic systems. Most systems have a cache hierarchy with L1, L2, etc. Instead of a "real" cache, there is the `SCR1NoCache` `SimObject`. This is a memory bus that connects the core and memory to retrieve, load, and write directly to the various registers of the SCR1. `SCR1NoCache` is a subclass of `NoCache` in gem5. Some functionalities have been modified to match SCR1, such as the removal of an MMU. Additionally, if the bus receives an instruction to read or write to a non-modelled address, it will read or write a null value. This allows testing numerous functionalities without worrying about having an ideal model.

<figure>
    {{
        image(
            url='img/mem.svg',
            transparent=true,
            no_hover=true
        )
    }}
    <figcaption>
        Graph representing the SCR1 core attached to memory
    </figcaption>
</figure>

```python
class scr1Memory(AbstractMemorySystem): # […]
    _parts = {
        "dmem": scr1MemoryPart(
            start=Addr(0x030_0000),
            size="32kB",
            latency=latency,
            latency_var=latency_var,
            bandwidth=bandwidth,
        ),
        "imem": scr1MemoryPart(
            start=Addr(0x020_0000),
            size="32kB",
            latency=latency,
            latency_var=latency_var,
            bandwidth=bandwidth,
        ),
        "rom": scr1MemoryPart(
            start=Addr(0x010_0000),
            size="32kB",
            latency=latency,
            latency_var=latency_var,
            bandwidth=bandwidth,
        ),
        "icn": scr1MemoryPart(
            start=Addr(0x000_0000),
            size="0x0000_EFFF",
            latency=latency,
            latency_var=latency_var,
            bandwidth=bandwidth,
        ),
        "mbox": scr1MemoryPart(
            start=Addr(0x000_F000),
            size="0x0000_0100",
            latency=latency,
            latency_var=latency_var,
            bandwidth=bandwidth,
        ),
        "bank": scr1MemoryPart(
            start=Addr(0x000_F800),
            size="0x0000_0200",
            latency=latency,
            latency_var=latency_var,
            bandwidth=bandwidth,
        )
    }  # […]
```
<figcaption>
    (Partial) definition of the memory Python class
</figcaption>

### Timers

The table below shows the registers used by a timer as defined in the RISC-V architecture of the SCR1.

<figcaption>
    Registers used by the SCR1 timer
</figcaption>

| Address             | Name         |
| ------------------- | ------------ |
| `TIMER_BASE` + 0x00 | `TIMER_CTRL` |
| `TIMER_BASE` + 0x04 | `TIMER_DIV`  |
| `TIMER_BASE` + 0x08 | `MTIME`      |
| `TIMER_BASE` + 0x0C | `MTIMEH`     |
| `TIMER_BASE` + 0x10 | `MTIMECMP`   |
| `TIMER_BASE` + 0x14 | `MTIMECMPH`  |

However, the timer requires more detailed modelling. The `MTIME`/`MTIMEH` registers represent the timer's counter value, which is incremented with each clock cycle. The `TIMER_CTRL` register contains a bit that enables or disables this increment. `TIMER_DIV` allows the counter to be incremented at a slower rate than the clock. Finally, the `MTIMECMP` and `MTIMECMPH` registers represent the value at which, when the combination of `MTIME` and `MTIMEH` exceeds this value, the timer sends a signal to the IT CTRL.

The modelling of these timers allowed the execution of another test firmware: a program named "PWM". This program uses the timer increments to generate a PWM by periodically writing to a register in the MailBox.

<figure>
    {{
        image(
            url='img/timer_system.svg',
            transparent=true,
            no_hover=true
        )
    }}
    <figcaption>
        Schema of the functional timer system
    </figcaption>
</figure>

### Connecting everything

We have the blocks, now we need to solder them together:

```python
# Creating the system where the model will run in
system = System()

system.clk_domain = SrcClockDomain()
system.clk_domain.clock = "50MHz"
system.clk_domain.voltage_domain = VoltageDomain()


# CPU
system.cpu = SCR1CPU()
system.cpu.createInterruptController()
system.cpu.createThreads()


# Memory Bus
proto_AHB = SystemXBar(width=32)
proto_AHB.badaddr_responder = BadAddr()
proto_AHB.default = proto_AHB.badaddr_responder.pio
system.proto_AHB = proto_AHB


# Connecting the CPU and the memory bus
system.cpu.icache_port = system.proto_AHB.cpu_side_ports
system.cpu.dcache_port = system.proto_AHB.cpu_side_ports


# Memory
system.mem_mode = "timing"

memory = scr1Memory()
for name, memoryPart in memory._parts.items():
    setattr(system, name, memoryPart)
    # Connecting the memory parts to the memory bus
    system.proto_AHB.mem_side_ports = getattr(system, name).port

system.mem_ranges = [part.range for part in memory._parts.values()]


# IPIC
system.cpu.ipic = scr1IPIC()

# Timers
system.timer1 = scr1Timer(pio_addr=0x0504000, it_code=7)
system.timer1.pio = system.proto_AHB.mem_side_ports


# Mailbox
system.mailbox = scr1MBox(pio_addr=0x000_F000, it_code=3)
system.mailbox.pio = system.proto_AHB.mem_side_ports


# System port
system.system_port = system.proto_AHB.cpu_side_ports

# Setup the workload to run
system.workload = RiscvBareMetal()

thispath = os.path.dirname(os.path.realpath(__file__))
elfie = os.path.join(
    "product-genx320-riscv/riscv_sdk/apps/hello_world/bin/hello_world.elf",
)

system.workload.bootloader = elfie


# Launching the simulation
root = Root(full_system=True, system=system)

m5.instantiate()


print(f"Beginning simulation!")
exit_event = m5.simulate()
```

### Results

To ensure the model functions correctly, it is necessary to run gem5 with the configuration file `scr1-fs.py`. This file initialises and connects all the elements required for the simulation of the SCR1. Among these elements, the binary file that the processor model should execute is essential. For the `hello_world` program, the following function is used:

```c
1 void print_hello_world() {
2     set_mbx_status_ptr(0xABCDEF);     // In the context of the internship,
                                        // the SCR1 was connected to a MBox
                                        // and was using FreeRTOS
3 }
```

To simulate scr1 executing `hello_world`, you must enter the following command:

```bash
$ ./build/RISCV/gem5.opt --debug-flags=Exec configs/scr1/scr1-fs.py
```

The debug flags allow you to see the RISC-V instructions executed in real-time. The following result is obtained:

```
system.cpu: 0x200200 @start    : csrrci zero, mstatus8
system.cpu: 0x200204 @start+4  : csrrwi zero, mie0
system.cpu: 0x200208 @start+8  : lui a0, 512
system.cpu: 0x20020c @start+12 : jalr zero, 528(a0)
system.cpu: 0x200210 @start_real  : auipc gp, 256
system.cpu: 0x200214 @start_real+4 : addi gp, gp, 1520
[...]
system.cpu: 0x200300 @main    : addi sp, sp, -32
system.cpu: 0x200304 @main+4  : lui a5, 513
[...]
system.cpu: 0x2003c4 @print_hello_world  : lui a0, 2749
system.cpu: 0x2003c8 @print_hello_world+4 : addi a0, a0, -529
system.cpu: 0x2003cc @print_hello_world+8 : jal zero, 4892
system.cpu: 0x2016e8 @set_mbx_status_ptr : addi a1, a0, 0
```

Here, we see the model successfully executing a `hello_world`.

Additionally, the interrupts are also functional:

```bash
$ ./build/RISCV/gem5.opt --debug-flags=FmtTicksOff,scr1MBox,Interrupt configs/scr1/scr1-wip.py
```

```
system.mailbox: Write 0xabcdef into STATUS_PTR
system.mailbox: Posting interrupt
system.cpu.interrupts: Interrupt 3:0 posted
```

Finally, to define the model's performance, it is interesting to compare the simulated time with the elapsed time on the host machine during the simulation:

| Simulated Time | Elapsed Time on Host |
| -------------- | -------------------- |
| 1 s            | 28.54 s              |
