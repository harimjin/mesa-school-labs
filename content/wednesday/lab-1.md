# Introduction

In the recent decade, it has become clear that the majority of massive stars are born in binaries or multiple-star systems. Throughout evolution, they will undergo binary interactions that can dramatically change their properties. One of the binary interactions is mass transfer, in which stars exchange mass and angular momentum. Mass transfer is thought to be responsible for various interesting stellar phenomena, such as different types of core-collapse supernovae, magnetic stars, X-ray sources, and gravitational wave sources. These phenomena are dependent on when mass transfer occurs and whether it is stable or unstable. In this lab, we will explore these across the initial binary parameter space - initial mass ratio and initial orbital period plane.

Stellar theorists call the initially more massive star as primary and the less massive star as secondary. (Caution: observers sometimes use primary to refer to the currently more massive star in a binary, which is not necessarily the initially more massive star.) The primary evolves faster and fills its Roche lobe and transfers mass onto the secondary. 



Task 1. Capture when mass transfer happens. 
Mass transfer can be divided into three cases, depending on the primary's evolutionary phase when mass transfer happens.
Case A is when the primary is doing core hydrogen burning.
Case B is when the primary is doing core helium burning.
Case C is when the primary is after core helium depletion.
Use this information to capture which case of mass transfer a system with 20Msun + 12Msun with an initial orbital period of 100 days undergoes, modifying src/run_binary_extras.f90.

TASK 1 block 

Use the following parameters
b% s1% center_h1
b% s1% center_he4
b% s1% center_c12
b% mtransfer_rate
Check the central mass fractions when the mass transfer happens (mass transfer rate exceeding 10^-10 Msun/yr). Print out "CaseA", "CaseB", "CaseC", depending on the conditions.
When you do a MESA run, do "rn | tee out.txt" to save the terminal output into a file.
"grep -ir Case out.txt" to print out the occurrences.

Hint1.
Check how core carbon depletion is captured below !!! HINT.

<details>
  <summary>Solution for Task 1</summary>
         !!!!!
         if ((b% s1% center_h1 > 1d-6) .and. (abs(b% mtransfer_rate/Msun*secyer) > 1d-10)) then
             write(*,*) '****************** CaseA ******************'
         else if ((b% s1% center_h1 < 1d-6) .and. (b% s1% center_he4 > 1d-6) .and. (abs(b% mtransfer_rate/Msun*secyer) > 1d-10)) then
             write(*,*) '****************** CaseB ******************'
         else if ((b% s1% center_he4 < 1d-6) .and. (abs(b% mtransfer_rate/Msun*secyer) > 1d-10)) then
             write(*,*) '****************** CaseC ******************'
         end if   
</details>
 

Hint1 :
Unfortunately, the units that MESA uses are not consistent. It is important to check the units of each parameter.
b% mtransfer_rate is in g/s. First, convert this into Msun/yr, then check if it exceeds 10^-10 Msun/yr.
secyr and Msun are a year in seconds and solar mass in grams. You can use it directly in run_binary_extras.f90.

Hint2:
How can you capture different burning phases?
You can capture the core hydrogen burning phase by "b% s1% center_h1 > 1e-6"
You can capture the core helium burning phase by "(b% s1% center_he4 > 1e-6) .and. (b% s1% center_h1 < 1e-6)"
You can capture the phase after core helium depletion by "(b% s1% center_he4 < 1e-6)"

Task 2. Determine whether it is a stable mass transfer or an unstable mass transfer
Sometimes, mass transfer becomes unstable as the secondary cannot accrete mass. There can be many ways to capture this, and one approximate way is to check the timestep. If the timestep gets extremely small (an order of seconds to minutes), it sometimes means that mass transfer is unstable. 
Use timestep 
b% s1% dt
When it is less than 40, terminate the evolution.

!!! TASK 2 here

Hint1: 
Check how to tell MESA to terminate the run in the HINT block.

Solution:
         if (b% s1% dt < 30) then
             write(*,*) '************************************************'
             write(*,*) '*** Terminated due to unstable mass transfer ***'
             write(*,*) '************************************************'
             extras_binary_finish_step = terminate
         end if   


Task 3. Run a model with random 
We will consider a primary mass of 20 Msun. 
First mass transfer case
Case A is followed with Case B
Initial secondary mass from 1 to 19 Msun
Initial orbital period in days 1 to 3000 days

Enter your result "Initial mass ratio (M2/M1)", "Initial orbital period in days", "Case", "Stable mass transfer?" in the following Google spreadsheet. If there was no mass transfer, leave "Case" and "Stable mass transfer" as blanks.
https://docs.google.com/spreadsheets/d/1HLwsGPu6w3t2NMUcdVYvkHFvqgIOUDkigfrZruN6Uo8/edit?usp=sharing
(Additional task: if you have many cores, you can perhaps try evolving both stars, download this file: and run.)


Acknowledgement: the MESA input files 
https://wwwmpa.mpa-garching.mpg.de/~jklencki/html/massive_binaries.html
