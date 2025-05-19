# Introduction

In the recent decade, it has become clear that the majority of massive stars are born in binaries or multiple-star systems. Throughout evolution, they will undergo binary interactions that can dramatically change their properties. One of the binary interactions is mass transfer, in which stars exchange mass and angular momentum. Mass transfer is thought to be responsible for various interesting stellar phenomena, such as different types of core-collapse supernovae, magnetic stars, X-ray sources, and gravitational wave sources. These phenomena are dependent on when mass transfer occurs and whether it is stable or unstable. In this lab, we will explore these across the initial binary parameter space.

# Task 1. Identifying different mass transfer cases
Mass transfer can be divided into three cases, depending on the evolutionary phase of the primary (initially more massive star) when the mass transfer happens. The primary evolves faster and fills its Roche lobe and transfers mass onto the secondary (initially less massive star). 
Case A is when the primary is doing core hydrogen burning.
Case B is when the primary is doing core helium burning.
Case C is when the primary is after core helium depletion.

We will use these criteria to capture which case of mass transfer occurs in a given system, modifying src/run_binary_extras.f90. This initial system is 20Msun (primary) + 12Msun (secondary) with an initial orbital period of 100 days. When you do a MESA run, do "rn | tee out.txt" to save the terminal output into a file.

We will use the following parameters in "extras_binary_finish_step" hook in run_binary_extras.f90.
b% s1% center_h1 ! Hydrogen mass fraction at the center of the primary
b% s1% center_he4 ! Helium mass fraction at the center of the primary
b% mtransfer_rate ! Mass transfer rate in g/s

First, think of a condition to capture the mass transfer phase, where the mass transfer rate exceeds 10^-10 Msun/yr.

<details>
  <summary>Hint 1</summary>
It is important to check the units of the parameters in MESA. In many cases, it is in cgs units, otherwise noted. b% mtransfer_rate is in g/s. First, let's convert this into the unit of Msun/yr. Variables "secyr" and "Msun" are a year in seconds and a solar mass in grams, respectively, that you can use directly in run_binary_extras.f90.
</details>

<details>
  <summary>Solution</summary>
  b% mtransfer_rate/Msun*secyer > 1d-10
</details>

Second, think of conditions to capture different burning phases:
1. Core hydrogen burning phase
2. Core helium burning phase
3. Core helium depletion onwards

<details>
  <summary>Hint 2</summary>
One can check the mass fractions of elements at the center. Check how core carbon depletion is captured in the "HINT" block. Check whether the mass fractions of hydrogen and helium are above or below 1e-6.
</details>

<details>
  <summary>Solution</summary>
You can capture the core hydrogen burning phase by "b% s1% center_h1 > 1e-6"
You can capture the core helium burning phase by "(b% s1% center_he4 > 1e-6) .and. (b% s1% center_h1 < 1e-6)"
You can capture the phase after core helium depletion by "(b% s1% center_he4 < 1e-6)"
</details>

Third, print out "CaseA", "CaseB", and "CaseC" in the terminal, depending on the different burning phases of the primary while mass transfer is happening. 

<details>
  <summary>Hint 3</summary>
  Check how to print a string in the terminal at core carbon depletion in the "HINT" block. 
</details>

<details>
  <summary>Solution for Task 1</summary>
         !!!!! TASK 1 block begins !!!
         if ((b% s1% center_h1 > 1d-6) .and. (abs(b% mtransfer_rate/Msun*secyer) > 1d-10)) then
             write(*,*) '****************** CaseA ******************'
         else if ((b% s1% center_h1 < 1d-6) .and. (b% s1% center_he4 > 1d-6) .and. (abs(b% mtransfer_rate/Msun*secyer) > 1d-10)) then
             write(*,*) '****************** CaseB ******************'
         else if ((b% s1% center_he4 < 1d-6) .and. (abs(b% mtransfer_rate/Msun*secyer) > 1d-10)) then
             write(*,*) '****************** CaseC ******************'
         end if   
         !!!!! TASK 1 block ends !!!
</details>



# Task 2. Determine mass transfer stability
Sometimes, mass transfer becomes unstable as the secondary cannot accrete mass. There can be many ways to capture this, and one approximate way is to check the timestep. If the timestep gets extremely small (an order of seconds to minutes), it sometimes means that mass transfer is unstable. 
Use timestep 
b% s1% dt
When it is less than 40, terminate the evolution.

!!! TASK 2 here

Hint1: 
Check how to tell MESA to terminate the run in the HINT block.


<details>
  <summary>Solution for Task 2</summary>
         !!!!! TASK 2 block begins !!!
         if (b% s1% dt < 30) then
             write(*,*) '************************************************'
             write(*,*) '*** Terminated due to unstable mass transfer ***'
             write(*,*) '************************************************'
             extras_binary_finish_step = terminate
         end if   
         !!!!! TASK 2 block ends !!!           
</details>
 



Task 3. Run a model with random 
We will consider a primary mass of 20 Msun. 
First mass transfer case
Case A is followed with Case B
Initial secondary mass from 1 to 19 Msun
Initial orbital period in days 1 to 3000 days

"grep -ir Case out.txt" to print out the occurrences.

Enter your result "Initial mass ratio (M2/M1)", "Initial orbital period in days", "Case", "Stable mass transfer?" in the following Google spreadsheet. If there was no mass transfer, leave "Case" and "Stable mass transfer" as blanks.
https://docs.google.com/spreadsheets/d/1HLwsGPu6w3t2NMUcdVYvkHFvqgIOUDkigfrZruN6Uo8/edit?usp=sharing
(Additional task: if you have many cores, you can perhaps try evolving both stars, download this file: and run.)


Acknowledgement: the MESA input files 
https://wwwmpa.mpa-garching.mpg.de/~jklencki/html/massive_binaries.html
