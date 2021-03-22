##BHJET

---------------------------------------------------------------------------------------------------------------------------------------

BHJET is a semi-analytical, multi-zone jet model designed for modelling steady-state SEDs of jets launched from accreting black holes. The key features of the model are: 
1) It is applicable across the entire black hole mass scale, from black hole X-ray binaries (both low and high mass) to active galactic nuclei of any class (from low-luminosity AGN to flat spectrum radio quasars),
2) It is designed to be more comparable than other codes to GRMHD simulations and/or RMHD semi-analytical solutions.
The model is fairly complex and fitting it to data is not always straightforward. As such, it is highly recommended to read this file carefully before running the code. It takes little time and will save you a lot of headaches later on.

---------------------------------------------------------------------------------------------------------------------------------------

The basic assumptions of the model are the following:
1) A certain amount of power is channeled from the inner accretion flow above the black hole and into the a nozzle region, which then starts accelerating into a jet. The nozzle is effectively a more physical description of a lamp-post corona. 
2) The jet accelerates along the z axis. Two modes of acceleration are available in the code: a pressure driven jet that accelerates due to a vertical gradient in pressure up to a moderate Lorentz factor (e.g. Crumley et al. 2016), and a magnetically-dominated jet in which Poynting flux is turned into bulk kinetic energy until the jet reaches an arbitrarily high Lorentz factor (Lucchini et al. 2019). The first case is typically used for modelling BHBs, the second for modelling highly relativistic AGN jets. Roughly, one can think of the first acceleration mode as being representative of a jet sheath, and the second mode as mimicking the spine usually observed in simulations (e.g. McKinney et al. 2006, Chatterjee et al. 2019) and/or some observations of AGN. Currently the two cases assume two different equipartition conditions to describe the magnetic field, electron and proton content of the jet. These are detailed in Crumley et al. 2016 and Lucchini et al. 2019 respectively.
3) At some distance from the black hole, a non-thermal population of particles appears. This is responsible for the bulk of the synchrotron radiation from radio up to (possibly) X-rays, as well as for the inverse-Compton bump observed in blazars.
4) Part of the adiabatic losses in the jet are offset by an unspecified acceleration mechanism. The power required for this mechanism to operate is not accounted for due to the complexities this would introduce in the calculations related to the jet dynamics. The balance between re-acceleration and adiabatic losses is described by: having variable temperature profile in the jet, solving the Fokker-Planck equation for the radiating leptons with a variable adiabatic timescale, and changing the percentage of particles that are accelerated into the non-thermal tail. The description of this assumption and their consequences are detailed in Crumley et al. 2016, Lucchini et al. 2019, Russel, Lucchini et al. in prep, as well as in the README file in the /lib folder. 
5) If required by the data, an optically thick, geometrically thin truncated disk can be included in the SED. The disk is parametrised by a truncation radius and a luminosity in Eddington units. Outside of the truncation radius, the temperature profile is given by the standard Shakura-Sunyaev description, and the disc scale h = H/R is assumed to be max(0.1,Ldisk), where Ldisk is the disk luminosity in Eddington units.
6) Additional components can be added to the SED if necessary. In particular, one can add a) an arbitrary external black body component (e.g. a contribution for the host galaxy), b) a simple broad line region or torus (for treating high accretion rate AGN), c) the stellar compantion of the compact objects (currently WIP and not in the repository). All of these photon fields, if included, also contribute to the inverse Compton emission.

---------------------------------------------------------------------------------------------------------------------------------------

The code has been re-written completely between 2018 and 2019 to improve in flexibility, performance and user-friendliness. The code is now based on a simple library, called High EneRgy astrophysics rEaSonable code librarY, or HERESY. It is contained and documented in the /lib folder. The classes in HERESY handle the particle distribution and radiation calculations through their methods. The structure of the code is straightforward, but there are a few important points:
1) The jet is arbitrarily divided into 80 zones (controlled by the integer variable nz) which do not interact with each other in any way. As the calculations move from the base of the jet to zmax, the grid which defines each zone changes. Up to 1000 Rg, the grid advances in steps of 2*r, where r is the radius of the jet at that particular distance. Outwards, the grid uses logarithmic steps instead. The reason for this choice (discussed in Connors et al. 2018) is that it prevents the final spectrum from being resolution-dependent. On top of that, the code runtime increases linearly with the zone count, so anything above ~70 will simply slow the code down for no gain.
2) The bulk of the code runtime is caused by the inverse Compton calculation, in particular for cases of moderate to high optical depth (>~0.05-ish). The code uses two tricks to improve the efficiency of the radiation calculations (which is particularly important, for example, for MCMC fits): a) the frequency grid over which the emission of each zone (both inverse Compton and synchrotron) is updated dynamically after calculating the appropriate scale frequencies, and b) more importantly, the inverse Compton itself is only calculated when it's expected to be bright enough to contribute meaningfully to the SED through the Compton_check function. IMPORTANT NOTE: this is set by the Compton_check function. If you are working on a faint source, particularly an AGN, with low Compton dominance (e.g. Sgr A*) you absolutely MUST ensure that this cutoff is not too aggressive in neglecting inverse Compton!

---------------------------------------------------------------------------------------------------------------------------------------

The free parameters of the model are (see both Lucchini et al. 2019 papers, and Russell, Luchcini et al. in prep, for a more in-depth discussion):

mbh 	- mass of the black hole. Always set before the fit.
Incl 	- viewing angle of the jet. Sets Doppler factor for the various regions. Always set before the fit.
dkpc 	- distance from source in kpc. Always set before the fit.
redshfit- self explanatory. Only used for modelling AGN and set before the fit.
jetrat	- amount of power injected at the base of the jet measured in Eddington units. Does not account for heating/acceleration of particles.
r\_0	- radius of the nozzle/corona, described by an outflowing, magnetized cylinder of radius r\_0 and height 2r\_0. Measured in units of r\_g.
z\_diss	- location of non-thermal particle injection region. sets optically thick to thin break, overall normalization and Compton dominance of non-thermal component, and for high accretion rate AGN sets the EC target field (BLR or BLR+torus). Measured in units of r\_g.
z\_acc   - if velsw > 1, sets location of the jet acceleration. Sets dependency of magnetic field with distance; smaller values correspond to faster dissipation before zacc (see velsw below). Measured in r\_g like zdiss. 
z\_max 	- maximum length over which jet calculations are carried out, in units of Rg. Typically frozen to large values in order to get a flat radio spectrum.
t\_e	- temperature of relativistic electrons in the nozzle/corona, expressed in kev.
f\_nth  - percentage of thermal particles accelerated into power-law tail. Typical values range between 10% and 90%. Typically frozen to 0.1.
f\_pl 	- reduces particle temperature and percentage of accelerated particles along the jet after zsh; both are decreased by a factor (log10(zdiss)/log10(z))^pldist. This results in an inverted radio spectrum. Set to 0 for a standard flat spectrum/isothermal jet.
pspec 	- slope of non-thermal particle distribution.
f\_heat 	- imitates shock heating; at z=zdiss bumps Te by a fixed factor heat, increasing the radiative efficiency of the jet after zsh. Typically set to 1 and frozen unless necessary.
f\_b	- sets the effective adiabatic cooling timescale, defined as t_ad = r/betaeff*c. This in turn sets the location of the cooling break in the radiating particle distribution. Typically kept free for blazars, otherwise set to 0.1 and frozen.
f\_sc  	- sets the maximum energy of non-thermal particles by parametrizing the acceleration timescale (and therefore acceleration efficiency). 
p\_beta - plasma beta (ratio between lepton and magnetic field energy density) at the base. If velsw = 0 or 1 this sets (and freezes) the equipartition value throughout the jet, if velsw > 1 this only sets the pair content and has to be frozen to a value high enough to have pair content ~approx unity (see infosw for how to check this). This is because the assumptions going into the calculation of the magnetic field when velsw>1 only hold if lepton energy density <<< (cold) proton energy density. If velsw > 1, this can be set to 0 to always enforce one proton per electron. See Crumley et al. 2016, Lucchini et al. 2019 for more details.
sig\_acc- only used if velsw > 1, sets the value for magnetization (defined as magnetic pressure plus energy density, over proton energy density plus lepton energy density and pressure, although the latter two are negligible). Sets Compton-y and peak frequencies for non-thermal bumps.
l\_disk - sets the luminosity of the disk in Eddington units, and the corresponding temperature is computed from knowing this luminosity and Rin.
r\_in 	- inner radius of the disk.
r\_out  - outer radius of the disk. Only has a minimal contribution to the SED, typically frozen. rout<=rin disables the disk.
compar1 - first parameter of the external photon field, depending on the value of compsw
compar2 - second parameter of the external photon field, depending on the value of compsw
compar3 - third parameter of the external photon field, depending on the value of compsw
compsw  - sets the external photon fields to be used in the IC calculation. 0 only includes SSC, 1 includes a uniform external black body contribution (e.g. a host galaxy), 2 includes the Broad Line Region and Torus of an AGN and ties their luminosity to that of the disk, 3 includes a stellar companion. If compsw = 1, compar1 is the temperature in Kelvin, compar2 is the total black-body luminosity, and compar3 is not used. If compsw = 2, compar1 is the fraction of disk photons reprocessed by the BLR, compar2 the fraction of disk photons reprocessed by the torus, and compar3 is not used. If compsw = 3, compar1 is the stellar temperature, compar2 is the stellar luminosity, and compar3 is the angle between the start and the jet axis in degrees. In this case, compar3 should be defaulted to 90 degrees unless there's good reason to do so.
velsw 	- This sets the jet velocity profile used by the code. If velsw = 0 or 1 the jet expands laterally and the resulting vertical gradient in pressure accelerates the jet (e.g. the profiles of Crumley et al. 2016), otherwise the jet base is highly magnetized and the jet accelerated by turning the Poyinting flux into bulk kinetic energy (Lucchini et al. 2019). In this case, the value of velsw is also the final Lorentz factor of the jet, achieved at a distance zacc (see above).
infosw  - information switch - the higher the value, the more info the code returns. 0 simply stores a total flux value in the photspec array; 1 also prints the total emission to, as well as contribution from each radiative component (e.g. thermal synchrotron, non-thermal synchrotron, thermal Comptonization, non-thermal IC emission, external fields, disk) to a file; 2 also prints the emission and particle distribution in each zone; 3, 4 and 5 print increasing amounts of information to the terminal.

Generally, no more than ~9 parameters should be fitted at the same time due to model degeneracies; depending on the application, many parameters can be frozen to a reasonable educated guess.

---------------------------------------------------------------------------------------------------------------------------------------

For running outside as a stand-alone code: 

Compile using:

./MakeBHJet

Input parameters can be changed in ip.dat in the Input directory. If you want to use a different file, change the name in jetwrap.cc and recompile. 

Run using:

./bhwrap.x

If you want to quickly plot the output, a simple python script is provided (Plot.py) and ran automatically by bhwrap.x. If you want to use your own plotting tools, this can be disabled by commenting out line 79 in the wrapper: 

system("python Plot.py");

You can also check the output of each zone in the jet as well as the lepton distribution along the jet; in order to do so change the parameter "infosw" to 2 and run Plot.py. Each zone will be plotted in different color; dark blue is close to the base, yellow is the outermost regions. Make sure infosw is set to 0 when fitting data to avoid slowing the code down, particularly on clusters that limit the amount of data that can be stored on text files.

---------------------------------------------------------------------------------------------------------------------------------------

For running inside ISIS:

./slirpAgnjet
make
make test

add in your .isisrc the following lines:
append_to_isis_load_path(path+"path/to/your/agnjet/folder");
append_to_isis_module_path(path+"path/to/your/agnjet/folder");

(on some systems if gsl libs aren't on obvious path you may need to
       explicitly include with -I and -L, but try this first!)

---------------------------------------------------------------------------------------------------------------------------------------