# Meeting-Cam
## Introduction  
A system to automate active speaker tracking in meetings using Acoustic Localization technique as its core and Computer Vision methods (Face Detection, Image Similarity) through OpenCV Library for accuracy enhancement.  

## Problem Statement and Description  
Organizations big or small have meetings and mostly these meetings are recorded for either reference sake for other usages such as detailed transcriptions. These recordings require the use of a professional camera operator and often individual microphones for all the speakers; such requirements are an unnecessary overhead and are often hard to meet. Imagine a scenario where a high-level meeting is
ongoing and in that meeting discussion is taking place on confidential information, having a cameraman to record such meetings would be absurd or would perhaps require the company to hire people (camera crew) in the house. Secrecy agreements with external organizations could be an option but again that is an overhead in terms of cost.  

Perhaps even a video conference could require a camera-man in case the meeting was between 1 to N or N to N clients. Meeting-Cam aims to provide an automated system of recording such meetings thus removing the need for having a cameraman to be physically present in the room. Meeting-Cam revolves around a device placed on the table that would be able to find the location of the sound and move the camera to point at that location.

## System Overview
Our system consists of a singular device that detects the sound and move towards the location of the sound. The device consists of an IP-Camera on which 2 Omni-Directional microphones are mounted. Microphones are there for capturing sound input. The device gets the input to the computer in which our algorithm runs as a Java application. The application is responsible for locating the sound source and rotating the camera towards it. Hereis the diagrammatic representation of our system.  
<p align="center">
<img src="https://github.com/Randhawa670/Meeting-Cam/blob/master/SystemOverview.PNG">
</p>

As you can see in the diagram above a meeting is being held and one person towards the right of the table is speaking and the camera is being pointed towards him. Now when another person will speak the camera will move towards him.

## System Architecture
<p align="center">
<img src="https://github.com/Randhawa670/Meeting-Cam/blob/master/SystemArchitecture.PNG">
</p>

The diagram above shows the high-level architecture of the system which will provide information about the flow of control in the system.

As can be seen in the diagram shown above the architecture is quite simple. The external world is a person who is currently speaking. This sound is detected by the microphones and is then passed onto the soundcard which stores it in its buffer. Now here our program comes in, it reads the data from the sound cards buffer and then our algorithm is run on it. We take samples of the sound of 62ms and perform the localization on that sample. Our system does also keep track of the previously calculated angle and uses it to adjust the newly calculated angle, this helps us in reducing any error or ignoring a wild change of angle. It also smoothes the movement of the camera. Now our program instructs the camera to move certain degrees so that it points at the location of the person.

The most imortant part of our system is Acoustic Localization or Sound Source Localization. It is also discussed in detailed in the next section.

## Sound Source Localization 
Sound Source localization is the most critical decision part of our system as the functionality of the whole system depends on it. There is four common type of approaches that could be used for detecting the sound’s source.

### Approaches
The four common type of approaches is Inter Aural Time Difference(ITD), Inter Aural Level Difference(ILD), Beam Forming(BF) and Microphone Directivity(MD). Let's discuss all four one by one and in the end, choosen approach is discussed. In the subsequent section, Algorithm for the implementation of the chosen approach is also discussed.

#### Inter-Aural Time Difference (ITD)  
The most common sound source localization technique is inter aural time difference (ITD). Also known as synonyms for this technique are Inter aural Phase Difference (IPD) or Time Difference of Arrival (TDA or TDOA). ITDs caused by the different propagation times as the sound wave travel from the source to both ears. For e.g. a source near to the left, the sound wave will reach the left ear slightly before it reaches the right ear. This time differece between two signals gives the estimate of the direction of the sound source.

#### Inter Aural Level Difference (ITD)  
Interaural level differences (ILD) are caused by the acoustic "shadow" of the head. For e.g. a source to the left, the sound wave will arrive at the left ear slightly louder than at the right ear. The ILD is completely based on the relative energy difference between two signals of a microphone pair. In contrast to most other techniques the phase of the received signals are of no importance and can, therefore, be neglected. Another often heard synonym for this technique is Inter-Aural Intensity Difference (IID).

#### Beam Forming (BF)  
Beam forming (BF) is a widely applied and well-known technique. It is based on inter-Aural Time differences and therefore strongly related to the classical ITD approach. The main difference to ITD is that there is no exact calculation but a position estimation by directing the beam former through space and look up for the maximal output.

#### Microphone Directivity (MD)  
Another approach is to use highly directional microphones for sound localization. Each microphone covers a specific region of the room. By looking for the maximal signal received by the microphones one can locate the sound source within the corresponding sector. Additional structures as shown in the chapter may be used for better results. Another similar approach may be to have the highly directional microphones movable and having them scan the room for a sound source. Then its position can be computed based on the geometrical information available from the aligned microphones. These approaches exist but the problem with these ideas is that resolution is generally low.  

#### Chosen Approach
Of all the approaches stated above available for sound source localization, inter-aural time differences are by far the easiest to use in a technical implementation. The only parameter that has to be known is the distance between the microphones (as long as there is no obstruction between them). 
Inter aural level differences, on the other hand, require some sort of artificial "head" between the microphones. This setup then has to be calibrated, i.e., the head related transfer function (HRTF) and ILD generated by a specific setup have to be measured, which is a rather tedious and elaborate process. Furthermore, any change in the artificial "head" or the microphones entails a recalibration. 

### Algorithm
The most famous algorithms that use inter-aural time difference (ITD) as its core are Dual Delay-line and correlation in the time domain. Basically, both are based on Jefferson Model that convert the Inter-Aural time difference to the azimuthal angle to the sound source. Let's first discuss the Model and then we will discuss both the algorithms. For our final system, we selected Dual Delay-line as the core algortihm for sound source localization. We selected it because in our tests we found that, it is more robust and gave the better estimation of the sound source.

#### The Jeffress Model
Jeffress model is one of the most successful models for explaining the processing of inter aural time differences in the brain.
In 1948, L. A. Jeffres proposed a hypothetical model of how neurons in the brain could make use of these tiny time differences, illustrated in the figure below:
<p align="center">
<img src="https://github.com/Randhawa670/Meeting-Cam/blob/master/jeffressmodel.PNG">
</p>

It consists of two major features: axonal delay lines and coincidence detector neurons. The coincidence detector neurons will (simply put) only fire if both of their inputs are excited simultaneously. Due to the finite running times of action potentials, the axons leading to the coincidence detectors act as delays.

If for example, a sound source is positioned to the left of the head, the action potentials from the left auditory nerve will have time to travel further along the upper (in the picture) axon, before the action potentials from the right ear arrive. Somewhere on the right side of the structure, they will simultaneously excite a coincidence detector neuron, which will then fire. In this way, the interaural time difference has been converted to a location in a neuronal structure.

#### Dual Dealy-Line
One of the implementations of the Jeffress model is the dual delay-line algorithm. It performs coincidence detection through comparing the shifted phases of the input signals in the frequency domain.

#### Correlation in the Time Domain
Another implementation of the Jeffress model is the simple cross correlation of the input signals recorded from the two microphones. The cross-correlation is a measure of the similarity between two signals (in our case L and R). It works by shifting the signals against each other by the time lag. This approach is also part of the human listening system through which we detect the direction of the sound's source.

## Demo

[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/YOUTUBE_VIDEO_ID_HERE/0.jpg)](https://www.youtube.com/watch?v=Qe0EuPyM0Fk&feature=youtu.be)


 
