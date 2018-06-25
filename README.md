# preCICE-adapter for the CFD code ANSYS Fluent
Developed by Bernhard Gatzhammer , Richard Hertrich tried to make an Update

1. How to build the Fluent-preCICE adapter: 
* go to the cloned repository

* Create a subdirectory lnamd64/2ddp/ for serial execution
    and for parallel execution add a /2ddp_host/ and /2ddp_node/.
    
    EDIT: I added these directories to the repository
* Put the libraries of preCICE and python2.7 (libpython2.7.so) into the lnamd64 folder.
* Adapt user.udf line 1: There are several main udf files
    - fsi_udf.c: For FSI simulations. Needs fsi.c.
    - wave_profile_udf.c: For wave simulations with inflow profile. Needs 
      wave_profile.c.
    - wave_maker_udf.c: For wave simulations with moving wall. Needs 
      wave_maker.c.
    - fsi_and_wave_profile_udf.c: For FSI + wave simulations. Needs fsi.c and 
      wave_profile.c.
    The variable SOURCES needs one one main udf file and the correspoding .c files.
* Type: make "FLUENT_ARCH=lnamd64" to start the build. Add a "clean" to clean it.

--------------------------------------------------------------------------------

2.1 How to start Fluent with GUI
- start the binary "fluent" from your simulation folder
- set double precision, processing options (serial or parallel) and the dimension
(for parallel select also "show more", "parallel settings", "mpi types" -> open mpi

2.2 How to start Fluent without GUI
- serial:   fluent 2ddp -g < steer-fluent.txt
- parallel: fluent 2ddp -g -t4 -mpi=openmpi < steer-fluent.txt
  (-t4 sets 4 processes for computations)
  (steer-fluent.txt drives Fluent, can also be done by hand)

3. Preparing a Fluent .cas file for UDF function usage:

- Copy the lnamd64 udf lib folder to the simulation folder into /libudf/
- Start Fluent and open the .msh or .cas file to be used
- In Fluent, go to top menu Define->User Defined->Functions...->Manage... and 
  load the libudf folder. 
  
  IT WORKS UP TO THIS POINT AT THE MOMENT
  
  The names of the udf functions should appear in the 
  Fluent command line window. (only necessary  if not yet included in the .cas file)
- Define a function hook at Initialization. (see udf manueal, define -> user defined -> function hooks)
- Define 1 user defined memory for the faces. (define -> user defined -> memory -> 1; adds one additional double to each face for precice face ids)

--------------------------------------------------------------------------------

4. Preparing a Fluent .cas file for FSI simulations

- Perform the steps in 1.,2. and 3.
- Set a user defined mesh motion according to function "gridmotions".
- When the parabolic inflow profile is needed, you have to read it from file 
  parabolic.prof, using the boundary conditions dialog box. Check, whether the
  number of points in parabolic.prof is way more than the elements at the inflow
  and that the velocities in the file are correct.

--------------------------------------------------------------------------------

5. Preparing a Fluent .cas file for Wave simulations

Go through the menus on the left and perform the following steps:
- General: Set transient simulation and gravity in y-direction = -9.81
- Models: Set Multiphase VOF explicit with Implicit body force formulation activated
- Materials: Air and water-liquid (from Fluent Database) are needed
- Phases: Name phases "phase-1-air" and set air as material, as well as 
  "phase-2-water" with water as material
- Cell Zone Conditions: Activate in operating conditions "Specified operating 
  density" and keep default value.
- Boundary Conditions:
  + Set the top boundary to pressure outflow
  + Set the left boundary to velocity inlet and set for Phase mixture the 
    x- and y- velocity components to follow the corresponding UDFs, for 
    Phase-2-water set the volume fraction UDF
- Dynamic Mesh: 
  + Set the structure boundary to follow the UDF
  + Set the domain to be deforming/remeshing as needed
- Reference values: Only needed for writing output (since Fluent usese 
  dimensionless values)
- Solution methods: 
  + Set the PISO (pressure implicit) scheme and as volume fraction scheme the 
    Modified HRIC scheme

Preparing the preCICE XML configuration:
- The number of valid digits for the timestep length needs to be 8, since fluent
  has only a single precision timestep length. Set this in the tag <timestep-length ... valid-digits="8"/>
