# Alexandru Mihai Hau
Jun 2021 - September 2021

# Research Project on Total Body PET Scanner

This project is about comparing the sensitivity and signal-to-noise ratio of a Positron Emission Tomography (PET) Scanner at different lengths (those used nowadays in industry, of around 10-15cm length, and the Total Body PET Scanners at 2m length). There are two steps involved in the project, both of them of great importance. For the first simulation, a small beam on positive and negative x-axis has been considered, with angular deviation of 180 degrees and the additional gaussian distribution of mean 0 and standard deviation 0.25 degrees. By locating the hits on detector, the purpose of this step is to check that computer simulations match the theory. Moreover, this step is important for future work in spatial resolution. The second step of the project involves creating random radiation distribution in order to simulate the sensitivity and SNR parameters.

## Requirements for running the project

The following software needs to be instaled on the station (Linux is the most preferred choice):

* **CMake** library written for C++
* **GEANT4 (GEometry ANd particle Tracking)** Software CERN Package for conducting the simulations. The platform is written in C++ and ran with the OPEN_GL Graphics Visualization Tool. The installing instructions can be found [here](https://geant4.web.cern.ch/support/getting_started).
* **ROOT** Software CERN Package for Data Analysis, written in C. ROOT instructions are found on  [this website](https://root.cern/install/).


## Geometry definition

A cyllinder has been simulated as a PET detector. The cyllinder has been chosen at two lengths: 15 cm and 2 meters. A patient has also been placed inside. The patient can be commented out in the BasicDetector.cc file, by removing the lines of introducing the G4PhysicalVolume patient object into the physical world. The material from which the patient is made can also be changed. For the first step, the detector should be declared very thick (30 cm thickness is a good choice), and the patient should be commented out. For the second part, thickness should be smaller, in order to put a hyperlinkconsider high-energy beams which escape the detector.

## Primary Generator Action

This file is used for creating the outgoing beam which represents the radiation inside the human body from the positron-electron annihilation event. Two gamma rays of 511 keV are created. Moreover, this is the file where the two steps of the problem are implemented. The gammas' energies - as well as the fact that the user works with gamma rays - are declared using the following line:

```
fParticleGun->SetParticleDefinition(particleTable->FindParticle(particleName="gamma"));
fParticleGun->SetParticleEnergy(511*keV);

```
**Note:** The two parts of the project are highly dependent on the code in BasicPrimaryGeneratorAction.cc file. For the **first** part, the beams are emitted on the same axis at 180 degrees angular deviation on the x-axis. For the second beam, an additional gaussian spread has been added through the following command line:

```
G4double gauss_value = twopi * G4RandGauss::shoot(0,0.25) / 360;
G4ThreeVector photonAntiDir = G4ThreeVector(std::cos(gauss_value), std::sin(gauss_value) * std::cos(theta),
  					        std::sin(gauss_value) * std::sin(theta));
``` 
Moreover, the origin of the radiation is set to 0 and remains constant throughout the simulation run. This part of the project has been implemented to check that all the components of the code yield an expected result. Moreover, this exercise has set the foundation for later improvements in the spatial resolution of the scanner.

```
G4double x0  = 0*cm, y0  = 0*cm, z0  = 0*cm;
fParticleGun->SetParticlePosition(G4ThreeVector(x0,y0,z0));

```
For the **second** part of the project, the origin of the outcoming photons is randomly placed inside the human phantom at each step. The random implementation has been done to recreate as much as possible the process of radiotracer injection in the human body and its random trajectory evolution. Beam origin generation has been done through the implementation of cyllindrical polar coordinates, as highlighted below:

```
G4double r = PET_radius * G4UniformRand();
G4double z = z_max * G4UniformRand() - 0.5 * z_max;
G4double alpha = alpha_max * G4UniformRand();

G4ThreeVector radiationOrigin = G4ThreeVector(r * std::cos(alpha), r * std::sin(alpha), z);
```
For the random origin generator, the efficiency of the PET Scanner has been calculated at the lengths of 15cm and 2m which can be varied in the BasicDetectorConstructon.cc file.

## Run action

The beginning of the simulation is conducted here. The file invokes the histograms and plots which are to be analysed. Moreover, in the Run Action file, after terminal execution, the sensitivity of the scanner and the SNR value are printed out. 

## Sensitive Detector hits

The hits on the sensitive detector are analysed on the BasicPETSD(Sensitive Detector abbreviation) file. Each event consists of more hits from the primary gammas and the secondary generated particles due to the photoelectric effect, Brehmstrahlung, annihilation event, etc. Each hit has the energy deposited and worked out from the following command line:

```
auto edep = step->GetTotalEnergyDeposit();

```

## Event Action

This file represents the methods implemented at the end of each event. Any event with deposited energy greater than 0.9 MeV is considered a good event (an event which helps in reconstructing the location of the positron - electron annihilation event). Each event consists of a collection of hits (G4HitsCollection), whose parameters can be accessed through BasicPetHit.cc and BasicPetHit.hh files. Warning: Do not change the AddEdep method of the hits collection. At the end of each event, the desired parameters are added into the histogram for event collection of the run

## Results

The following results have been worked out:
* The sensitivity of the PET-Scanner increases 30 times when the length is shifted from 15cm to 2m. Research papers from UC Davis, Harvard and other various institutes claim a 40-fold increase. However, the implemented geometry model has been simple and straightforward. Different materials need to be assigned to the human phantom, and more complex geometrical shapes need to be assigned
* The SNR value is of 0.25 approximately, yielding a NECR number of roughly 0.33. These are, again, values in agreement with the current Nuclear PET-Medicine Research.
* The impact of gamma ray non-collinearity on the spatial resolution of the scanner has been calculated and estimated to approximately 90cm. This represents a huge error, hence improvements need to be conducted in this topic. 
