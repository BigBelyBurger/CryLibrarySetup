# Cry Library Setup

## Installing and creating CRY

### Creating the necessary folders
First of all, create a new folder where you want the cry libary to be. This can easily be done using the command :
```
mkdir (FOLDER_NAME)
```

### Installing CRY
Inside this folder, create a "work" folder and download the source file available at https://nuclear.llnl.gov/simulation/, and save it in your new folder. Once you have saved the source code, run:
```
tar -xzf cry_v1.7.tar.gz
rm cry_v1.7.tar.gz
```
If all goes well, you should have a new folder "cry_v1.7" (or a higher number if a new version of CRY is released)

### Setting up the correct paths and sources
Go into the cry_v1.7 folder and type:
```
export G4WORKDIR=../work
make
```
You now have two options. you can either download and setup a new version of Geant4 (complicated and annoying), or use a university version (only possible if you have a university server with Geant4 downloaded).
Inside your Geant4 instalation, you should have four files, "bin", "include", "lib64" and "share". Go into bin and run:
```
source geant4.sh
```
Then, go into lib64/Geant4_YOUR_INSTALLATION_VERSION/geant4make/ and run:
```
source geant4make.sh
```
Please note that the above two commands must be run every time you start working on Cry (or Geant4). I strongly recommend making a bash file that can easily be run to make things easier.

Go back into your cry installation and edit your "setup.sh" file. You should see two uncommented lines:
```
export CRYHOME=$1
export CRYDATAPATH=$1/data
```
These should be edited to be:
```
export CRYHOME=FOLDER_NAME/cry_v1.7/
export CRYDATAPATH=FOLDER_NAME/cry_v1.7/data/
```
And run:
```
source setup.sh
```
### Our first Make
In your main cry_v1.7 folder. Go into /cry_v1.7/geant/ and type "make". You should get errors now, which we will work to remove one by one. The errors you will get may be different than mine, but should be explained well enough to be removed on your own. Here are the problems I faced.

## Setting up Cry for Geant4

### Removing deprecated files and functions (G4Elastic.hh: No such file or directory")
My first error is "src/PhysicsList.cc:144:10: fatal error: G4LElastic.hh: No such file or directory". This can be fixed by removing the lines:
```
#include "G4LElastic.hh
// elastic scattering
G4HadronElasticProcess* theElasticProcess =
                              new G4HadronElasticProcess;
G4LElastic* theElasticModel = new G4LElastic;
theElasticProcess->RegisterMe(theElasticModel);
pmanager->AddDiscreteProcess(theElasticProcess);
```
I then have an error "src/PhysicsList.cc:146:10: fatal error: G4LENeutronInelastic.hh: No such file or directory". Same procedure once more, remove the lines:
```
#include "G4LENeutronInelastic.hh"
// inelastic scattering
G4NeutronInelasticProcess* theInelasticProcess =
                           new G4NeutronInelasticProcess("inelastic");
G4LENeutronInelastic* theInelasticModel = new G4LENeutronInelastic;
theInelasticProcess->RegisterMe(theInelasticModel);
pmanager->AddDiscreteProcess(theInelasticProcess);
```
And finally, "src/PhysicsList.cc:148:10: fatal error: G4LCapture.hh: No such file or directory", remove the lines:
```
#include "G4LCapture.hh"
// capture
G4HadronCaptureProcess* theCaptureProcess =
                         new G4HadronCaptureProcess;
G4LCapture* theCaptureModel = new G4LCapture;
theCaptureProcess->RegisterMe(theCaptureModel);
pmanager->AddDiscreteProcess(theCaptureProcess);
```

### Warning "altitude" error
Typing "make" once more should provide a new list of errors (hooray!). First of all, the line " warning: declaration of ‘altitude’ shadows a member of ‘CRYSetup’ [-Wshadow]" is simply a warning a can completely be ignored. 

### Unit errors ('MeV was not declared in this scope)
You should now have an error about "src/PrimaryGeneratorAction.cc:135:49: error: ‘MeV’ was not declared in this scope", or other units such as "m", "cm", etc... You should go into the files quoted, in this case, it would be "PrimaryGeneratorAction.cc" then go to the line specified, here line135, and change "UNIT" to "CLHEP::UNIT", in our case, "MeV" would become "CLHEP::MeV".

### Particle Iterator error ('theParticleIterator' was not declared in this scope)
At one point, you should also have an error "src/PhysicsList.cc:158:3: error: ‘theParticleIterator’ was not declared in this scope". To fix this, go into "PhysicsList.cc", and change the line:
```
theParticleIterator->reset();
```
to:
```
auto theParticleIterator = GetParticleIterator();
theParticleIterator->reset();
```

### Setting up the work and cosmic file path
Finally, edit the "test_cry" file. First of all, the line "source /usr/gapps/cern/geant4.9.0/setup" should link directly to your "setup.sh" file. This can be done using the line:
```
source ../setup.sh
```
And the line "bin/$G4SYSTEM/cosmic cmd.file | tee geant4.9.0.out" must reference the "cosmic" file, which can be found in the work folder we created at the beggining. The line then becomes:
```
../work/bin/Linux-g++/cosmic cmd.file | tee output.txt
```
If you do not want to be spammed by Cry tests, simply comment out the "echo" and "dif" lines.

### Done ?
Once all the errors have been fixed, you should be set and can run the Cry Library ! Congratulations, crack yourself a fresh beer, you deserve it.

## A few helpfull tips for running Cry
