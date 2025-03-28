+++
authors = ["Lucien Gheerbrant"]
title = "Wearable IoT device for fitness tracking and monitoring"
description = "A focus on SpO2, electronic devices and packaging"
date = 2024-10-28
[markdown]
numbersections = true
[taxonomies]
tags = ["Electronic", "Microelectronic", "IoT", "Fitness Tracker", "Packaging"]
[extra]
toc = true
katex = true
+++

# Wearable IoT device for fitness tracking and monitoring

Lucien GHEERBRANT
25433424

## 1. Introduction

Over the last few decades, electronic miniaturisation has reached unprecedented levels. What once required an entire lab now fits in our pocket as a smartphone. Likewise, the miniaturisation of sensors has transformed bulky medical equipment into devices like smartwatches. Instead of being confined to a hospital, patients can now receive real-time diagnoses while leading normal lives. Health tracking has also extended beyond medical care into fitness with the rise of smartwatches, rings, and other wearables. Sensors are now everywhere, and for good reason. These wearable devices are part of an integrated IoT ecosystem that tracks fitness activities and health metrics.

One key technology driving this growth is the $\mathrm{SpO}\_{2}$ sensor, which monitors blood oxygen levels in real time. Found in many fitness trackers, this sensor is critical for both health and fitness applications. As health and wellness awareness grows, these advancements allow medical research to gather unprecedented data, leading to deeper insights into the human body.

## 2. State-of-the-art

### 2.1. Pulse Oximetry

$\mathrm{SpO}\_{2}$ sensors are commonly used in wearable devices to measure blood oxygen saturation. These sensors work using a method called pulse oximetry, which calculates the percentage of oxygenated hemoglobin in the blood. The $\mathrm{SpO}\_{2}$ value is defined by

$$
\mathrm{SpO}\_{2} = \frac{\mathrm{HbO}\_{2}}{\mathrm{HbO}\_{2} + \mathrm{Hb}}
$$

with $\mathrm{HbO}\_{2}$ and $\mathrm{Hb}$ the concentrations of oxygenated and non-oxygenated hemoglobins, respectively [1].

Have you ever seen a smartwatch lighting up an LED on its back? This is the first step in measuring $\mathrm{SpO}\_{2}$, where light from the LED passes through the skin and reflects back, allowing the device to detect changes in blood properties.

There are different approaches to pulse oximetry, as shown in Figure 1, with classical devices (e.g., fingertip pulse oximeters) that directly measure light transmission through a thin body part like the finger (Figure 1a). In comparison, wearable devices like smartwatches use a reflective method, where the light is reflected back from the skin to a sensor (Figure 1b).

<figure>
    {{
        image(
            url='img/Finger_Pulse_Oximeter.webp',
            transparent=true        )
    }}
    <figcaption>Fig. 1a: Classical oximetry.</figcaption>
</figure>



<figure>
    {{
        image(
            url='img/apple_watch_red.webp'
        )
    }}
    <figcaption>Fig. 1b: Apple Watch oximetry.</figcaption>
</figure>

*Figure 1: Professional VS. consumer $\mathrm{SpO}\_{2}$ tracking.*

The measurement is based on using two different wavelengths of light, typically red and infrared, which are absorbed differently by $\mathrm{HbO}\_{2}$ and $\mathrm{Hb}$. By applying the Beer-Lambert law, which relates the concentration of a substance to the amount of light absorbed, it is possible to determine the relative levels of oxygen in the blood. This technique is widely used due to its ease of implementation and noninvasive nature, requiring only simple interaction between the device and the user via light rays.

The calculation of $\mathrm{SpO}\_{2}$ relies on a ratio $R$, which is derived from the absorption of red (660 nm) and infrared (905 nm) light by oxygenated and deoxygenated hemoglobin. $R$ is approximately $\frac{\varepsilon(660 \, \text{nm})}{\varepsilon(905 \, \text{nm})}$, with $\varepsilon(\lambda)$ the extinction coefficients of hemoglobin at the $\lambda$ wavelength (red and IR in this case). Using this $R$, we can obtain:

$$
\mathrm{SpO}\_{2} = \frac{0.81 - 0.2 \times R}{0.09 \times R + 0.73}
$$

These extinction coefficients are measured by shining light through the body and analyzing how much is absorbed by the blood. By using sensors placed on the skin, the device can detect how much light passes through the tissue. The more oxygen in the blood, the less red light is absorbed, and the more infrared light is absorbed.

From this point, LEDs (for emitting light) and photodiodes (for detecting it) are critical. Those two optoelectronics components will be the main focus of this report.

<figure>
    {{
        image(
            url='img/ppg-side-view.webp',
            no_hover=true,
            transparent=true
        )
    }}
    <figcaption>Fig. 2: Photoplethysmography (PPG) sensor operation.</figcaption>
</figure>

Figure 2 illustrates the basic operation of a PPG sensor used in wearable devices for measuring $\mathrm{SpO}\_{2}$. The system consists of LEDs and photodiodes integrated within the device, along with a CPU or microcontroller that processes the data. During measurement, the LEDs emit light (typically red and infrared) onto the skin, where it passes through organic tissue and blood. The photodiodes then detect the reflected light that has interacted with the blood [2]. The changes in light absorption are related to the varying concentrations of oxygenated and deoxygenated hemoglobin in the blood. These variations are captured as a signal by the photodiodes, which is then processed by the embedded system to calculate blood oxygen saturation. This process enables non-invasive monitoring, where the interaction between the light and tissue provides valuable health information without any direct contact with the blood itself.

### 2.2. PDs and LEDs on the same package

Previously, in earlier designs of wearable sensors, LEDs and photodiodes were often packaged together in a single unit [3]. This approach made the overall design more compact and streamlined, reducing the complexity of the PCB layout and saving space in devices like smartwatches. By housing both components together, manufacturers could simplify the assembly process and align the LED and photodiode more easily to detect reflected light. This integration helped minimize light leakage, allowing for a more controlled environment inside the sensor, which at the time, was thought to enhance performance. Additionally, the smaller footprint was especially beneficial for making wearables more lightweight, more "wearable".

<figure>
    {{
        image(
            url='img/ADPD144RI.webp'
        )
    }}
    <figcaption>Fig. 3: Packaging of the ADPD144RI sensor [4].</figcaption>
</figure>

Figure 3 illustrates an example of this two-in-one package in a compact sensor module, as seen in the ADPD144RI, developed by Analog Devices [4]. The ADPD144RI was specifically designed for use in wearable devices such as fitness trackers and smartwatches, where reducing space and power consumption is essential. Its packaging is optimized for PPG applications like $\mathrm{SpO}\_{2}$ and heart rate monitoring, ensuring reliable performance in compact environments. The module integrates two LEDs (660 nm red and 880 nm IR), along with four photodiodes in an array. The same photodiodes are used to detect light from both wavelengths, with the LEDs activated alternately to avoid interference and ensure accurate detection of light after it passes through tissue.

<figure>
    {{
        image(
            url='img/LED-Electronic.webp'
        )
    }}
    <figcaption>Fig. 4a: LED mechanism.</figcaption>
</figure>

<figure>
    {{
        image(
            url='img/photodiode-electronic.webp'
        )
    }}
    <figcaption>Fig. 4b: Photodiode mechanism.</figcaption>
</figure>

*Figure 4: Core components for $\mathrm{SpO}\_{2}$ sensors. [1]*

Figure 4 shows the basic principles behind an LED (Figure 4a) and a photodiode (Figure 4b), which are key components in $\mathrm{SpO}\_{2}$ sensors. In the LED, light is produced at the p-n junction, where electrons from the n-type region combine with holes in the p-type region. As these electrons drop from the conduction band to the valence band, they release energy as photons. The colour of the emitted light is determined by the bandgap energy ($E_g$) of the material used in the LED.

The photodiode, on the other hand, works in reverse bias mode. Here, light that hits the p-n junction generates electron-hole pairs. The depletion region at the junction becomes wider in reverse bias, which makes the photodiode more sensitive to incoming light. When photons with enough energy strike the junction, they excite electrons from the valence band to the conduction band, creating free electrons and holes. These charge carriers are then separated by the electric field in the depletion region, producing a photocurrent that matches the intensity of the light hitting the photodiode.

<figure>
    {{
        image(
            url='img/table.webp',
            no_hover=true,
            transparent=true
        )
    }}
    <figcaption>Fig. 5: Popular materials for LEDs and PDs in $\mathrm{SpO}\_{2}$ sensors.</figcaption>
</figure>

Table 1 presents the materials commonly used for LEDs and photodiodes in $\mathrm{SpO}\_{2}$ sensors. Selecting appropriate materials is crucial for achieving the specific wavelengths necessary for accurate oxygen saturation measurements [5].

Infrared LEDs emitting wavelengths greater than 760 nm typically use GaAs or AlGaAs due to their suitable bandgap energies for efficient infrared light emission. In contrast, red LEDs operating within the 610–760 nm range often employ GaP or GaAsP, which emit light in the visible red spectrum essential for differentiating between oxygenated and deoxygenated hemoglobin. Photodiodes responsible for detecting both red and infrared light are commonly made from Si, as it can detect a broad range of wavelengths while maintaining low power consumption.

While Analog Devices has not disclosed the exact materials used in their sensor, it is likely that the 660 nm red LED is made from GaP or GaAsP and the 880 nm infrared LED from GaAs or AlGaAs, given their widespread use for these wavelengths. The photodiodes are also probably made from silicon, considering its effectiveness in detecting both visible and near-infrared light and its suitability for low-power, integrated microelectronic applications.

Wearable sensor technology has evolved from integrating LEDs and photodiodes within the same package to physically separating them on the device, as described in.

### 2.3. The issue with packaging LEDs and PDs together

However, integrated designs faced significant limitations. The close proximity of the LED and the photodiode led to direct light leakage, where emitted light reached the photodiode without passing through the skin tissue [3]. This resulted in inaccurate PPG readings because the light had not interacted sufficiently with the blood flow necessary for reliable measurements of $\mathrm{SpO}\_{2}$ and heart rate. The integrated sensors were also more susceptible to electronic and optical interference, compromising signal quality and device performance. Despite engineering efforts, these drawbacks outweighed the benefits, leading to a decline in the use of integrated designs.

Manufacturers shifted to separating the LEDs and photodiodes to enhance measurement accuracy and reliability [3]. This separation ensures that more emitted light penetrates the skin and interacts with blood flow before detection, reducing optical crosstalk and improving the signal-to-noise ratio. The design allows for better control over the optical path, minimizing motion artifacts and ambient light interference. Although this approach introduces greater design complexity and may increase production costs, it significantly improves sensor performance and reliability.

<figure>
    {{
        image(
            url='img/watch-ifixit-sensor.webp'
        )
    }}
    <figcaption>Fig. 6: Apple Watch sensors [6].</figcaption>
</figure>

For example, the Apple Watch (Figure 6) employs separated LEDs and photodiodes, utilizing multiple wavelengths to achieve high-precision $\mathrm{SpO}\_{2}$ and heart rate monitoring. Advanced fabrication techniques enable seamless integration of these components within a sleek design. Targeting consumers seeking premium health monitoring features, the device justifies its higher price point with enhanced accuracy, durability, and safety features like water resistance and robust materials.

With separate packaging, optical isolation is often used to prevent unwanted light leakage between the LEDs and photodiodes, ensuring that the photodiode primarily detects light that has interacted with blood flow rather than direct emissions from the LED. This is achieved by using opaque barriers or optical coatings that block stray light, further enhancing the signal-to-noise ratio. Additionally, separate packaging allows for the use of different encapsulation materials for the LEDs and photodiodes, tailored to optimize each component's performance; for example, materials that enhance light emission for the LED, and coatings that increase sensitivity or reduce ambient light interference for the photodiode.

Another advantage of separate packaging is the ability to customize the placement and orientation of the LEDs and photodiodes. This flexibility enables designers to position the components at optimal angles and distances, allowing for better light penetration into the skin and reducing motion artifacts. Additionally, these techniques can incorporate protective layers and moisture barriers to enhance the durability and longevity of the sensors, ensuring reliable performance even under varying environmental conditions.

### 2.4. Wrist-based vs. finger-based measuring

Wrist-based $\mathrm{SpO}\_{2}$ measurements offer significant advantages in terms of comfort and convenience, allowing for continuous monitoring without the need for bulky fingertip sensors. This approach is especially appealing for wearable devices like smartwatches that prioritize mobility and ease of use.

However, several limitations impact the accuracy of wrist-based $\mathrm{SpO}\_{2}$ monitoring. The wrist has lower blood perfusion compared to the fingertip, resulting in a weaker PPG signal and less reliable readings. Additionally, the anatomical complexity of the wrist, with its bones and soft tissues, can obstruct light transmission, causing signal attenuation [7].

Moreover, the wrist is a highly mobile joint, making it prone to motion artifacts during everyday activities. Movements such as bending or flexing the wrist can introduce noise and affect the stability of the readings. Lastly, technical constraints in wrist-worn devices, such as smaller sensor sizes and lower power output, further limit their measurement quality compared to fingertip pulse oximeters, which remain the preferred choice for clinical applications [8].

### 2.5. Skin Colour

Skin colour affects the accuracy of $\mathrm{SpO}\_{2}$ measurements due to variations in melanin concentration, which influences how light is absorbed by the skin. Darker skin tones with higher melanin levels absorb more light at the red and infrared wavelengths used in pulse oximetry. This reduces the amount of light that penetrates the skin and reaches the blood vessels, resulting in a weaker PPG signal and potentially less reliable readings [7].

Most $\mathrm{SpO}\_{2}$ algorithms are calibrated using data from people with lighter skin tones, leading to systematic biases when applied to individuals with darker skin. This can cause inaccurate $\mathrm{SpO}\_{2}$ estimates, especially in cases of low blood oxygen saturation. Efforts to address these issues include adaptive algorithms that account for melanin levels and using additional wavelengths to improve measurement accuracy across different skin tones. However, accuracy discrepancies remain, underscoring the need for improved calibration standards and further research [7].

### 2.6. Flexible electronics

Flexible electronics represent a significant advancement in the field of wearable health monitoring, including devices for measuring $\mathrm{SpO}\_{2}$. Unlike traditional rigid components, flexible electronics utilize bendable materials such as polymer substrates or thin metal foils, enabling sensors to conform to curved surfaces like the human body. This flexibility allows for more comfortable and reliable contact, essential for accurate measurements, especially when placed on non-flat areas such as the wrist or forearm [2], [9].

The use of organic semiconductors, thin-film transistors, and flexible printed circuits has facilitated the development of lightweight and stretchable sensor arrays. These materials can endure mechanical stress without compromising electrical performance, which is critical for wearable devices subject to continuous movement. Additionally, flexible electronics enable more strategic placement of components, such as separating LEDs and photodiodes in $\mathrm{SpO}\_{2}$ sensors to reduce interference and improve accuracy [9], or having a finger-based sensor in a wearable way (as shown in Figure 7).

<figure>
    {{
        image(
            url='img/ramuz.webp'
        )
    }}
    <figcaption>Fig. 7: Flexible PPG / $\mathrm{SpO}\_{2}$ finger-based sensor. [9]</figcaption>
</figure>

As flexible electronics continue to evolve, they offer promising opportunities for innovative wearable designs that can measure a range of biometric data while maintaining user comfort and enhancing sensor integration in everyday health-monitoring devices.

## 3. Perspectives

Although wearable $\mathrm{SpO}\_{2}$ technology has advanced significantly, it remains unsuitable for professional medical use due to its limitations in accuracy and reliability. Current wrist-based oximetry lacks the precision needed for clinical applications, as factors such as skin colour, ambient light, and motion artifacts can significantly impact readings. For the general public, $\mathrm{SpO}\_{2}$ monitoring in smartwatches and fitness trackers tends to be more gadget-like, offering basic insights rather than delivering reliable, health-critical data. These devices often serve more as wellness features than as accurate medical tools.

A more suitable application for wearable $\mathrm{SpO}\_{2}$ technology is in sports and athletic training, where continuous monitoring can help track oxygen saturation during physical activities. Athletes could use wearable $\mathrm{SpO}\_{2}$ devices to get real-time insights into their cardiovascular performance. However, the technology should be used under the guidance of an expert, such as a coach or sports scientist, who can interpret the data correctly. Given that wearable sensors may require calibration adjustments for factors like skin tone, expert oversight is essential to ensure accurate use and interpretation.

Another practical use case is in high-altitude training or mountaineering, where athletes can benefit from monitoring $\mathrm{SpO}\_{2}$ to detect early signs of hypoxia or altitude sickness. While wearable devices can offer valuable information on oxygen saturation trends, the measurements should not be solely relied upon for critical decision-making. Instead, they can be used as supplementary tools, with calibration adjustments and regular cross-referencing with professional-grade pulse oximeters to ensure data accuracy.

The limitations of current wearable $\mathrm{SpO}\_{2}$ devices could be addressed through the use of flexible electronics and smart clothing equipped with multiple measuring points. By integrating flexible sensors across different body parts, smart clothing could provide a more comprehensive view of blood oxygen levels, improving measurement accuracy and reducing the effects of motion artifacts. Such an approach would be particularly beneficial in the athlete monitoring scenario, where data from multiple locations could be aggregated for more reliable $\mathrm{SpO}\_{2}$ tracking. With advancements in sensor technology and data processing, this multi-point approach could potentially make wearable $\mathrm{SpO}\_{2}$ monitoring accurate enough for more professional applications in sports and fitness.

## 4. Conclusions

The development of $\mathrm{SpO}\_{2}$ sensing technology for wearables highlights a significant advancement in electronic components designed for non-invasive health monitoring. The main components that enable $\mathrm{SpO}\_{2}$ measurement are LEDs and photodiodes, which work together using PPG to detect blood oxygen levels. Through controlled light emission (usually red and infrared) and precise light detection, these components allow wearables to measure oxygen saturation by analyzing light absorption in the blood.

The state of the art in wearable $\mathrm{SpO}\_{2}$ technology has improved significantly, driven by advancements in miniaturization and integrated circuitry. Recent developments include the shift toward separate packaging of LEDs and photodiodes to reduce direct light reflection and enhance measurement accuracy, as well as the exploration of flexible electronics to improve wearability. However, while these advancements have improved accuracy and user comfort, they remain limited by factors like motion artifacts, skin tone variability, and lower blood perfusion at the wrist compared to the fingertip.

In terms of limitations, wearable $\mathrm{SpO}\_{2}$ sensors often struggle with maintaining accurate readings due to environmental interference and power constraints. Wrist-based devices, for instance, face challenges from anatomical complexity and motion sensitivity, which interfere with signal quality. Additionally, size and power limitations in wearable designs often result in smaller, less powerful LEDs and photodiodes, reducing the overall reliability of the measurements, particularly in professional or clinical settings.

Future perspectives include advancements in flexible and stretchable electronics, which could enable multi-point measurement and improve signal stability during motion. These technologies may eventually allow for more accurate and robust $\mathrm{SpO}\_{2}$ measurements across different body sites, moving wearables closer to medical-grade reliability. Additionally, the integration of smart textiles with distributed sensors could enhance measurement accuracy, making wearables more suitable for athletic performance tracking and professional health monitoring. Overall, continuous improvements in sensor technology and fabrication methods are essential for wearable $\mathrm{SpO}\_{2}$ sensors to reach their full potential as reliable tools in health monitoring.

## References

[1] J. G. Webster, *Design of Pulse Oximeters*. CRC Press, 1997. doi: 10.1201/9780367802592.
[2] W.-C. Kuo, T.-C. Wu, and J.-S. Wang, "Design and Application of a Flexible Blood Oxygen Sensing Array for Wearable Devices," *Micromachines*, vol. 13, no. 10, p. 1742, Oct. 2022, doi: 10.3390/mi13101742.
[3] K. B. Kim and H. J. Baek, "Photoplethysmography in Wearable Devices: A Comprehensive Review of Technological Advances, Current Challenges, and Future Directions," *Electronics*, vol. 12, no. 13, p. 2923, Jul. 2023, doi: 10.3390/electronics12132923.
[4] [Online]. Available: https://www.analog.com/media/en/technical-documentation/data-sheets/ADPD144RI.pdf
[5] N. Morita and W. Iwasaki, "Design and Fabrication of a Thin and Micro-Optical Sensor for Rapid Prototyping," *Sensors*, vol. 23, no. 17, p. 7658, Sep. 2023, doi: 10.3390/s23177658.
[6] "Apple Watch Teardown." Accessed: Oct. 29, 2024. [Online]. Available: https://www.ifixit.com/Teardown/Apple+Watch+Teardown/40655
[7] Z. Zhang and R. Khatami, "Can we trust the oxygen saturation measured by consumer smartwatches?," *The Lancet Respiratory Medicine*, vol. 10, no. 5, pp. e47-e48, May 2022, doi: 10.1016/s2213-2600(22)00103-5.
[8] R.-J. Shei, I. G. Holder, A. S. Oumsang, B. A. Paris, and H. L. Paris, "Wearable activity trackers—advanced technology or advanced marketing?," *European Journal of Applied Physiology*, vol. 122, no. 9, pp. 1975-1990, Apr. 2022, doi: 10.1007/s00421-022-04951-1.
[9] J. V. Dcosta, D. Ochoa, and S. Sanaur, "Recent Progress in Flexible and Wearable All Organic Photoplethysmography Sensors for SpO2 Monitoring," *Advanced Science*, vol. 10, no. 31, Sep. 2023, doi: 10.1002/advs.202302752.