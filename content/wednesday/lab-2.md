# Common Envelope evolution with MESA

# General ideas

In this part of the lab, we explore how we can model the common envelope (CE) phase of binary stars using MESA. We use MESA's single star module `star` to evolve the donor star. The effect of the companion star is modeled by using several `other_*`-hooks in the `run-star-extras.f90` file.

![CE cartoon](/wednesday/CE_cartoon.png)

**Fig. 1**: Cartoon illustrating the 1D CE method. The companion (black dot) is spiraling inside the giant star. The black and blue arrows indicate the relative velocity of the companion and the drag force respectively. The heating zone is highlighted by the purple ring (taken from [Bronner et al. (2024)](https://doi.org/10.25518/0037-9565.12322))

We initiate the CE run by placing the companion star of mass $M_2$ at an orbital separation $a_\mathrm{ini}= 0.99R_1$ where $R_1$ is the radius of the donor star. At this point, the companion star is subject to dynamical friction, which leads to loss of angular momentum and orbital energy, leading to a decrease in the orbital separation. We can model the strength of the drag force by using the formulation from [Ostriker (1998)](https://ui.adsabs.harvard.edu/link_gateway/1999ApJ...513..252O/doi:10.1086/306858)
$$
F_\mathrm{drag} = \frac{4\pi (G M_2)^2\rho}{v_\mathrm{rel}^2} I,
$$
where $G$ is the gravitational constant, $v_\mathrm{rel}$ is the relative velocity between the donor star and the companion star, $\rho$ is the density of the donor star at the location of the companion star, and $I$ is the Coulomb logarithm. For subsonic motion, the Coulomb logarithm is given by
$$
I_\mathrm{subsonic} = \frac{1}{2} \ln \left( \frac{1 + \mathcal{M}}{1 - \mathcal{M}} \right) - \mathcal{M},
$$
where $\mathcal{M} = v_\mathrm{rel}/c_s$ is the Mach number and $c_s$ is the sound speed. For supersonic motion, the Coulomb logarithm can be approximated as
$$
I_\mathrm{supersonic} = \frac{1}{2} \ln \left( \frac{1}{1 - \mathcal{M}^2} \right) + \ln \left( \frac{2 a}{r_\mathrm{min}} \right),
$$
where $a$ is the orbital separation and $r_\mathrm{min}$ relates to the size of the companion star. 

Now that we know the strength of the drag force, we can calculate the change in orbital energy $E_\mathrm{orb}$ caused by the drag force. The change in orbital energy over one timestep $\Delta t$ is given by
$$
\Delta E_\mathrm{orb} = F_\mathrm{drag} v_\mathrm{rel} \Delta t.
$$
The change in orbital energy can be related to the change in orbital separation by
$$
\Delta E_\mathrm{orb} = -\frac{G M_{1,a} M_2}{2 a} + \frac{G M_{1,a} M_2}{2 a^\prime} = -\frac{G M_1 M_2}{2} \left( \frac{1}{a} - \frac{1}{a^\prime} \right),
$$
where $a^\prime$ is the new orbital separation, assuming that $M_{1,a}$ is roughly constant and that the orbit stays circular. Thus, we can model the evolution of the orbital separation.

The back reaction of the drag force on the donor star is modeled by using the `other_energy`-hook that allows us to modify the internal energy (heating/cooling). Because we know how much orbital energy is removed by the drag force (see above), we can add exactly the same amount of energy as heat in the envelope of the donor star. We heat all the layers within one accretion radius $R_\mathrm{a}$ of the companion star, with
$$
R_\mathrm{a} = \frac{G M_2}{v_\mathrm{rel}^2}.
$$
Additionally, we use a Gaussian weighting kernel $\propto \exp[-(\Delta r/R_\mathrm{a})]$ to have a smooth heating profile, where $\Delta r = |r - a|$.


# Tasks for students

1. **Check out the `run-star-extras.f90` file**: Please download the provided MESA directory from [here](/wednesday/lab2_part2.zip). This inlcudes may file, most of which you can ignore for now. Have a close look at the `src/run-star-extras.f90` file, especially the `other_energy` hook and the `extras_finish_step` function. Try to understand how the drag force is calculated and how it is used to update the orbital separation.

{{< details title="Solution" closed="true" >}}
The drag force is calculated in lines 339 and the orbital separation is updated in line 345. We are making use of the `xtra(i)` variables in the `star_info` pointer. These are particularly handy as we do not have to worry about things going wrong, if MESA decides to do a `retry`.

All of the heating is done in the `CE_heating` function at the end of the file.
{{< /details >}}


1. **Run the CE model**: Run the CE model with the provided `inlist` file. You are provided with a $12\,\mathrm{M}_\odot$ red supergiant model (from the `12M_pre_ms_to_core_collapse` test suite) and a $1.4\,\mathrm{M}_\odot$ companion star (could be a neutron star). Everything is already implemented as described above. You can ignore all the inlists expect for `inlist_CE`. The other inlists are taken from the test suite and not modified. So you really just have to do `./mk && ./rn`. Have a look at how the orbital separation changes over time. The orbital separation is directly printed to the terminal but also saved to the `history.log` as `separation`. Use the [MESA explorer](https://billwolf.space/mesa-explorer/) to visualize `separation` vs `star_age` (you need to upload your `history.log`file).

{{< details title="Solution" closed="true" >}}
The orbital separation is $\sim 40.7 \, {\rm R}_\odot$ after 2 years of CE evolution.
{{< /details >}}

3. **Change the companion mass**: Run the same setup but vary the mass of the companion star. What happens if you increase the mass of the companion star? What happens if you decrease it? How does this affect the orbital separation? We have tested the cases for $ 0.5\,\mathrm{M}_\odot \leq M_2 \leq 2.0\,\mathrm{M}_\odot$. Depending on the companion mass, you might need to adjust the stopping criterion in the `inlist_CE` file.

{{< details title="Hint" closed="true" >}}
Have a close look at the `inlist_CE` file. Try to spot the `x_ctrl` variable that corresponds to the companion mass.
{{< /details >}}

{{< details title="Solution" closed="true" >}}
For more massive companions, the orbital separation is larger. When visualizing the orbital evolution over time, the more massive companion plunges in faster compared to less massive companion. 
{{< /details >}}

4. **(Bonus task) Modify the drag force**: The current implementation of the drag force is based on the assumption that the companion star is moving on a straight path through a uniform density background. This is not the case during the CE phase. In a more realistic scenario, the drag force may be weaker. Implement a free parameter in the drag force calculation that allows you to scale the drag force by a factor $C_\mathrm{drag}$. Implement it such that you can control this factor from the `inlist_CE` file. What happens if you set $C_\mathrm{drag} = 0.5$? Is this what you expected? 

{{< details title="Hint" closed="true" >}}
You might want to define a `x_ctrl` variable in the `inlist_CE` file that you can use a global pre-factor for the drag force. Try to locate the line where the drag force in computed in the `run_star_extras.f90`.
{{< /details >}}

{{< details title="Solution" closed="true" >}}
Update the `inlist_CE` file like this:
```fortran
&controls
    ...
      x_ctrl(10) = 1.0d0  ! drag factor parameter
    ...
/ ! end of controls namelist
```

Then update the `run_star_extras.f90` as follows:
```fortran
Fdrag = s% x_ctrl(7) *  4*pi*rho_r*(G * M2 / vrel)**2 * I
```
You can find a full implementation ==here==.

For $C_\mathrm{d}<1$ the plunge-in takes longer and the separation afterwards is a little larger. This is expected as the drag force is generally weaker.

{{< /details >}}

1. **(Bonus task) Modify the drag force prescription**: Let's extend the drag force prescription to include the density gradient of the envelope. Implement the drag force prescription from [MacLeod & Ramirez-Ruiz (2015)](https://doi.org/10.1088/0004-637X/803/1/41) in the `run-star-extras.f90` file. This prescription is only valid for supersonic motion. Implement this drag force prescription into the `run_star_extras.f90`.

{{< details title="Hint" closed="true" >}}
{{< /details >}}

{{< details title="Solution" closed="true" >}}
{{< /details >}}

1. **(Bonus task) Implement common envelope ejection**: The current implementation of the CE phase does not allow for mass loss. In this task, you wil implement a mass loss prescription to follow the dynamical mass ejection in the CE phase.


{{< details title="Hint" closed="true" >}}
{{< /details >}}

{{< details title="Solution" closed="true" >}}
{{< /details >}}


# Note on assumptions, limitations and points to improve
- CE is not point-symmetric (not 1D)
- such simulations are only valid for low mass ratios, i.e., $M_2/M_1 \ll 1$
- drag force only valid of straight line motion uniform density background
- there exist other drag force prescriptions that take the density gradient into account ([MacLeod & Ramirez-Ruiz 2015](https://doi.org/10.1088/0004-637X/803/1/41)) or circular motion ([Kim & Kim 2007](https://doi.org/10.1086/519302))
- no mass loss in this CE simulation, therefore no mass CE ejection possibility
- no angular momentum transfer from companion to envelope