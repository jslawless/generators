---
title: "1 - Matrix Element Generator"
teaching: 20
exercises: 40
questions:
- "What are Monte Carlo Generators?"
- "Why are we using simulated samples in CMS?"
- "How are simulated samples created in CMS?"
objectives:
- "Use the MadGraph generator in standalone mode and get familiar with the basic syntax"
- "Analyze the produced LHE files"
keypoints:
- "MadGraph is a widely used tool to generate matrix-element predictions for the hard scatter for SM and BSM processes."
- "Standalone MadGraph can run interactively on-the-fly or by importing the predefined text scripts"
- "Gridpacks are used for large scale productions  with consistency guaranteed"
- "LHE level information is not physical and parton shower is needed to describe full physics"
---

# Introduction and first steps

Although quite old, [link](https://arxiv.org/pdf/1304.6677.pdf) is a great reading material to get a general overview of Monte Carlo event generators.
Monte Carlo event generators are essential components of almost all experimental analyses and are also widely used by theorists and experiments to make predictions and preparations for future experiments.
It is one of the topics where we CMS experimentalists and theorists have the closest connections to, theorists give us predictions and experimentalists verify them with the actual data.
Although Monte Carlo event generators are extremely important tools in HEP, they are often used as black boxes which we more or less treat them as "data".
Our aim is to get the minimal background of how these tools are working and analyze them using the generator level information.

Samples that are used by CMS experiments go through several steps of simulation :
1. Monte Carlo event generator
2. Detector simulation
3. Pileup mixing
4. Trigger emulation
5. Object econstruction

We focus on "1. Monte Carlo event generator" in this tutorial.
Monte Carlo event generator can be further divided into several subpieces as each steps can be factorized and can be handled through separate calculations :
1. Parton distribution function (PDF)
2. Hard scattering (matrix element calculation)
3. Parton shower & hadronization
First of all, LHC is a proton-proton collider, hence we need information on how partons (quarks and gluons) are distributed in the proton (PDF).
Hard scattering is the part where calculations can be treated perturbatively, interactions of incoming partons with the largest momentum transfer (usually the physics process we are interested in).
Parton shower & hadronization further describes how the particles involed in the hard scattering evolve, working downwards to lower momentum scales even to a point where perturbative calculations break down.


## Using Standalone Madgraph

In the first part of the exercise, we will use the matrix element generator MadGraph5 _aMC@NLO, or in short MadGraph [link](https://launchpad.net/mg5amcnlo).
MadGraph can perform the calculations for many different physics processes (both SM and BSM) at leading and next-to-leading order (LO & NLO) in QCD.
Because of its easy user interface and flexibility with UFO models, you can test wide variety of physics modeling.
We will now first see how MadGraph runs interactively in standalone mode using simple `W+` (wplus) process as an example.

We will first use the interactive prompt of MadGraph to generate proton proton collision events that produce W bosons.
First, log in to a new session on the LPC cluster (`ssh -Y <USERNAME>@cmslpc-el8.fnal.gov`).
Make sure you have completed the <a href="../setup.html">setup</a> steps!
Then, start the interactive prompt of Madgraph:
~~~bash
cd ~/nobackup/cmsdas_2025_gen/MG5_aMC_v3_5_2/
./bin/mg5_aMC
~~~
{: .source}

Madgraph is configured and steered through text-based cards.
The process definitions can be stored in a card called `proc_card.dat`.
You can look at an example using the following command:
~~~bash
!cat wplustest_4f_LO_proc_card.dat
~~~
{: .source}

Note: the exclamation mark is used to execute shell commands within Madgraph, e.g. `!cat` in the above example.
~~~bash
import model sm-ckm
#switch to diagonal ckm matrix if relevant for speed
#import model sm-lepton_masses

define ell+ = e+ mu+ ta+
define ell- = e- mu- ta-

generate p p > w+, w+ > ell+ vl @0

output wplustest_4f_LO -nojpeg
~~~
{: .output}

**Copy/paste the commands line-by-line and pay attention to the output.**

The two most important lines of this block are the model import (`import model sm-ckm`) and the instructions on the process to generate (`generate p p > w+, w+ > ell+ vl`).
Within the MG directory you can find a directory `models`, that contains different pre-installed models.
The most obvious one that we are using in the example is `sm` - the standard model at leading order in perturbative QCD.
Model parameters can be configured through ''restriction cards'', in this example `restrict_ckm.dat` loaded through the syntax `sm-ckm`.
This specific restriction card uses a non-diagonal [CKM matrix](https://en.wikipedia.org/wiki/Cabibbo–Kobayashi–Maskawa_matrix) (diagonal CKM is the default otherwise for simplification and faster running).
One great feature of MadGraph is it's flexibility in terms of physics models to use.
To generate a sample using a new physics model one can use the UFO interface. A database of models can be found in the [feynrules model database](http://feynrules.irmp.ucl.ac.be/wiki/ModelDatabaseMainPage).

The practically most relevant part is that MG figures out all relevant Feynman diagrams contributing to a process.
If you are trying to set up a new MC sample, looking at these Feynman diagrams is a great way to check that you actually get the physics you want.
To check them out you can open the individual plots in e.g. `wplustest_4f_LO/SubProcesses/P0_qq_wp_wp_lvl/matrix*.ps` with `gv`, `display` or `evince`.
You can also use the `ps2pdf` program to convert the post script files into PDFs.

Alternatively, remove `-nojpeg` from the output line and look at the diagrams in jpeg format using `display`.

Now that Madgraph has figured out the feynman diagrams you can start the actual computation within the MG5 prompt with
~~~bash
launch
~~~
{: .source}

Hint: if you closed the interactive MG session for some reason you can still launch without rerunning the previous commands with
~~~bash
./bin/mg5_aMC
launch wplustest_4f_LO
~~~
{: .source}
Madgraph will ask you a few more questions. Press `tab` to turn off the timer (otherwise, MadGraph will move on by itself after 60 seconds).
~~~
/===========================================================================\
| 1. Choose the shower/hadronization program     shower = Not Avail.        |
| 2. Choose the detector simulation program    detector = Not Avail.        |
| 3. Choose an analysis package (plot/convert) analysis = Not Avail.        |
| 4. Decay onshell particles                    madspin = OFF               |
| 5. Add weights to events for new hypp.       reweight = Not Avail.        |
\===========================================================================/
~~~
{: .output}
The first one you can just skip by pressing \<RETURN\>. As we did not install any other `shower`, `detector`, `analysis package`, they are in `Not Avail.` state.

~~~
Do you want to edit a card (press enter to bypass editing)?
/------------------------------------------------------------\
|  1. param : param_card.dat                                 |
|  2. run   : run_card.dat                                   |
\------------------------------------------------------------/
 you can also
   - enter the path to a valid card or banner.
   - use the 'set' command to modify a parameter directly.
     The set option works only for param_card and run_card.
     Type 'help set' for more information on this command.
   - call an external program (ASperGE/MadWidth/...).
     Type 'help' for the list of available command
 [0, done, 1, param, 2, run, enter path][90s to answer] 
~~~
{: .output}


Let's take a look at the `param card` and see how the values are set, press `1` and `ENTER` (\<RETURN\>) to investigate the parameter settings.
~~~
###################################
## INFORMATION FOR MASS
###################################
Block mass
    5 4.700000e+00 # MB 
    6 1.730000e+02 # MT 
   15 1.777000e+00 # MTA 
   23 9.118800e+01 # MZ 
   25 1.250000e+02 # MH

...

###################################
## INFORMATION FOR DECAY
###################################
DECAY   6 1.491500e+00 # WT 
DECAY  23 2.441404e+00 # WZ 
DECAY  24 2.047600e+00 # WW 
DECAY  25 6.382339e-03 # WH 
~~~
{: .output}

Let's take a look at the `run card` and see how the values are set, press `2` and `ENTER` (\<RETURN\>) to investigate the run settings.
~~~
#*********************************************************************
# Number of events and rnd seed                                      *
# Warning: Do not generate more than 1M events in a single run       *
#*********************************************************************
  10000 = nevents ! Number of unweighted events requested
  0   = iseed   ! rnd seed (0=assigned automatically=default))

...

#*********************************************************************
# Collider type and energy                                           *
# lpp: 0=No PDF, 1=proton, -1=antiproton,                            *
#                2=elastic photon of proton/ion beam                 *
#             +/-3=PDF of electron/positron beam                     *
#             +/-4=PDF of muon/antimuon beam                         *
#*********************************************************************
     1        = lpp1    ! beam 1 type
     1        = lpp2    ! beam 2 type
     6500.0     = ebeam1  ! beam 1 total energy in GeV
     6500.0     = ebeam2  ! beam 2 total energy in GeV

...

#*********************************************************************
# Standard Cuts                                                      *
#*********************************************************************
# Minimum and maximum pt's (for max, -1 means no cut)                *
#*********************************************************************
 10.0  = ptl       ! minimum pt for the charged leptons
 -1.0  = ptlmax    ! maximum pt for the charged leptons
 {} = pt_min_pdg ! pt cut for other particles (use pdg code). Applied on particle and anti-particle
 {}     = pt_max_pdg ! pt cut for other particles (syntax e.g. {6: 100, 25: 50})

...

#*********************************************************************
# Minimum and maximum invariant mass for pairs                       *
#*********************************************************************
 0.0   = mmll    ! min invariant mass of l+l- (same flavour) lepton pair
 -1.0  = mmllmax ! max invariant mass of l+l- (same flavour) lepton pair
 {} = mxx_min_pdg ! min invariant mass of a pair of particles X/X~ (e.g. {6:250})
 {'default': False} = mxx_only_part_antipart ! if True the invariant mass is applied only
                       ! to pairs of particle/antiparticle and not to pairs of the same pdg codes.

...

#*********************************************************************
# maximal pdg code for quark to be considered as a light jet         *
# (otherwise b cuts are applied)                                     *
#*********************************************************************
 4 = maxjetflavor    ! Maximum jet pdg code
~~~
{: .output}

Try editting the beam energy (`ebeam1` and `ebeam2`) `6500` to `6800` as we are now running at 13.6TeV beam energy.
When done with editting, escape after saving the changes in the text file.

MadGraph allows you to change settings by interactively typing in below as well.
~~~
set run_card nevents 5000
~~~
{: .output}

Take a look at the run card again and see if number of events to generate (`nevents`) is changed to `5000`.
And change it back to `10000` using same command and check again.

As shown above, there are several phase space cuts set by default (e.g. `10.0 = ptl`).
There is a handy command that removes all phase space cuts at once (instead of doing `set run_card ptl 0`, `set run_card ptj 0`, ... one by one by hand).
~~~
set no_parton_cut
~~~
{: .output}

Take a look at the card again and see if lepton pt cut (`ptl`) is changed to `0`.
Keep in mind that the cuts you give before doing `set no_parton_cut` will be removed by this command.
So don't forget to do `set no_parton_cut` before giving the cuts you wish to give.


Once you are done, please provide the path to the pre-made run_card: `wplustest_4f_LO_run_card.dat`


What is the cross section determined by Madgraph?

> ## Obtaining the cross section
>
> Define a process (e.g. from the process card above) and launch.
>
> > ## Solution
> >
> > ~~~bash
> > launch wplustest_4f_LO
> > ~~~
> > {: .source}
> >
> > ~~~
> > === Results Summary for run: run_01 tag: tag_1 ===
> >
> >      Cross-section :   2.715e+04 +- 39.45 pb
> >      Nb of events :  10000
> > 
> > INFO: No version of lhapdf. Can not run systematics computation
> > store_events
> > INFO: Storing parton level results
> > INFO: End Parton
> > reweight -from_cards
> > decay_events -from_cards
> > INFO: storing files of previous run
> > INFO: Done
> > ~~~
> > {: .output}
> > The cross section calculated by MG is `2.715e+04 +- 39.45 pb`.
> {: .solution}
{: .challenge}

The main output that MG produces is called ''LHE file''.
The LHE file (Les Houches Event file) is a standard file format that stores process and event information from parton-level event generators.
The documentation can be found [here](https://arxiv.org/pdf/hep-ph/0609017.pdf).

In general, the LHE file contains a header with description of the settings of the generator (e.g. process and run information),
and multiple event blocks (one for each event).
The LHE file is plain text, so it's usually a good idea to use some compression algorithm to save space - MG zips the output by default.

> ## Looking at the LHE output
> Find the LHE file produced by MG and find the first event block.
> > ## Example solution
> > Exit MG, then do
> > ~~~bash
> > find -path './wplustest_4f_LO/*.lhe.gz'
> > gzip -d ./wplustest_4f_LO/Events/run_01/unweighted_events.lhe.gz
> > less ./wplustest_4f_LO/Events/run_01/unweighted_events.lhe
> > ~~~
> > {: .source}
> > 
> > ~~~
> > <LesHouchesEvents version="3.0">
> > <header>
> > ...
> > </header>
> > ...
> > <event>
> >  5      0 +2.7145900e+04 7.93095700e+01 7.54677100e-03 1.33102200e-01
> >         2 -1    0    0  501    0 +0.0000000000e+00 +0.0000000000e+00 +4.5829549845e+01 4.5829549845e+01 0.0000000000e+00 0.0000e+00 -1.0000e+00
> >        -1 -1    0    0    0  501 -0.0000000000e+00 -0.0000000000e+00 -3.4311969734e+01 3.4311969734e+01 0.0000000000e+00 0.0000e+00 1.0000e+00
> >        24  2    1    2    0    0 +0.0000000000e+00 +0.0000000000e+00 +1.1517580112e+01 8.0141519579e+01 7.9309573878e+01 0.0000e+00 0.0000e+00
> >       -13  1    3    3    0    0 -1.6845086581e+01 +2.2368564620e+01 -2.2614075432e+01 3.5993138689e+01 0.0000000000e+00 0.0000e+00 1.0000e+00
> >        14  1    3    3    0    0 +1.6845086581e+01 -2.2368564620e+01 +3.4131655543e+01 4.4148380890e+01 0.0000000000e+00 0.0000e+00 -1.0000e+00
> > <mgrwt>
> > <rscale>  0 0.79309574E+02</rscale>
> > <asrwt>0</asrwt>
> > <pdfrwt beam="1">  1        2 0.70507000E-02 0.79309574E+02</pdfrwt>
> > <pdfrwt beam="2">  1       -1 0.52787646E-02 0.79309574E+02</pdfrwt>
> > <totfact> 0.15019241E+05</totfact>
> > </mgrwt>
> > <rwgt>
> > <wgt id='1'> +2.3707085e+04 </wgt>
> > ...
> > </rwgt>
> > </event>
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

> ## What does each column mean?
>
> > ## Solution
> > `ID`, `status`, `mother1`, `mother2`, `color`, `anticolor`, `px`, `py`, `pz`, `E`, `mass`, `life time`, and `spin`
> > ~~~output
> > -11  1    3    3    0    0 -2.3393803385e+01 -7.4187481776e+00 -1.5274153214e+02 1.5470062541e+02 0.0000000000e+00 0.0000e+00 1.0000e+00
> > ~~~
> > {: .output}
> > This line tells you that a positron (`ID`) is an outgoing particle (`status`) with Z as its mother (`mother1` and `mother2` : 3rd particle is Z which is `ID=23`) with no color (`color` and `anticolor`), ...
> {: .solution}
{: .challenge}

> ## MadGraph syntax
> If you want to add another process, e.g. production of W- in the above example, you can add another process with `add process p p > w-, w- > ell- vl~`
>
> A detailed introduction to the syntax is given in [this documentation](https://cp3.irmp.ucl.ac.be/projects/madgraph/wiki/InputEx).
> Some very basic things to keep in mind:
>
> `generate p p > e+ e-` will generate any diagram that is compatible with the used model that produces an electron / positron pair
>
> `generate p p > e+ e- / Z` will exclude diagrams that contain a Z boson as internal paricle
>
> `generate p p > e+ e- $ Z` will exclude the Z boson from appearing in the s-channel (careful about gauge invariance)
>
> `generate p p > Z > e+ e-` will always include a Z boson in the s-channel (careful about gauge invariance)
>
> `generate p p > w+ QED=3` will include all QED contributions, otherwise the QED order is always set to its minimal value
{: .callout}

> ## Bonus: Obtaining the cross section of W boson production
>
> The exercise above only contained W+ bosons (only positive charge).
> Add production of the negatively charged W bosons and calculate the cross section.
> Before running MG, think about what result you would expect, i.e. by how much do you think the cross section should increase.
> Compare the results of W+ and W+/-. What do you conclude?
>
> > ## Solution
> >
> > ~~~bash
> > import model sm-ckm
> > 
> > define ell+ = e+ mu+ ta+
> > define ell- = e- mu- ta-
> > generate p p > w+, w+ > ell+ vl @0
> > add process p p > w-, w- > ell- vl~ @1
> > 
> > output wtest_4f_LO -nojpeg
> > launch 
> > ~~~
> > {: .source}
> >
> > When prompted about the run_card, use `wplustest_4f_LO_run_card.dat` again.
> > ~~~
> >   === Results Summary for run: run_01 tag: tag_1 ===
> > 
> >      Cross-section :   4.667e+04 +- 63.91 pb
> >      Nb of events :  10000
> > ~~~
> > {: .output}
> > The cross section calculated by Madgraph is `4.732e+04 +- 57.08 pb`.
> > While one would naiively expect the cross section to double by including W- bosons we only get a cross section that is ~40% larger.
> > The simplified explanation is that the initial state protons contain more up valence quarks than down valence quarks.
> {: .solution}
{: .challenge}


## Using the gridpack workflow

As mentioned previously, interactive running of MG5 is useful for developments and quick tests, but ultimately not practical for large-scale production.
To avoid having to use the interactive mode, one can make use the card structure of MG.
A fully automated workflow for running MG and producing gridpacks is maintained in the [genproductions repository](https://github.com/cms-sw/genproductions).
A gridpack is simply an archive file that contains all the executable MG5 code needed to produce LHE events for a given process, which can then be executed easily on many different grid workers (hence the name).
It has the advantage that once it is created, it is a one-button program to generate events, no thinking required.
In this part of the exercise, we will use the same input cards as before to create a gridpack, run it, and compare the results to before.

Gridpacks are generated using the gridpack_generation script, which we will run in local mode, i.e. on the machine we are currently logged in to.
Note that scripts are provided to run the gridpacks on other computing infrastructures such as the CERN batch system and CMSConnect, which is useful for more complicated processes.

To create a gridpack, we simply call gridpack_generation.sh and pass the process name and card location to it.

> ## Setting and unsetting the CMSSW environment
> You should not have activated a CMSSW environment in this exercise so far.
> However, if you did so before, you need to unset it in order to not interfere with the genproductions script.
> You can run the following command to unset the CMSSW environment, or log in to a clean new session.
> ~~~bash
> eval `scram unsetenv -sh`
> ~~~
> {: .source}
{: .callout}

We will be generating a gridpack with cards similar to the commands we've used in the standalone example above.
The cards are located in the MG section of the genproductions directory

~~~bash
cd ~/nobackup/cmsdas_2025_gen/genproductions_mg352/bin/MadGraph5_aMCatNLO
time ./gridpack_generation.sh wplustest_4f_LO cards/examples/wplustest_4f_LO local
~~~
{: .source}

> ## Naming conventions
> There are certain naming conventions for the input cards.
> For a given process name $NAME, the input cards must be named as $NAME_run_card.dat, $NAME_proc_card.dat, etc...
{: .callout}

<!-- The cards for Run2 UL samples that were generated with MG can be found in `bin/MadGraph5_aMCatNLO/cards/production/2017/13TeV/` of the [genproductions repo](https://github.com/cms-sw/genproductions/tree/master/bin/MadGraph5_aMCatNLO/cards/production/2017/13TeV). -->


~~~bash
mkdir work
cd work
tar xf ../wplustest_4f_LO_el9_amd64_gcc11_CMSSW_13_2_9_tarball.tar.xz

NEVENTS=10000
RANDOMSEED=12345
NCPU=1
./runcmsgrid.sh $NEVENTS $RANDOMSEED $NCPU
~~~
{: .source}

This will produce an output LHE file called `cmsgrid_final.lhe`.

## Comparing the LHE output

There are multiple ways of analyzing an LHE file, each of which has its own advantages and disadvantages.
For the purpose of this exercise, we will use a pre-made pyroot script. 
~~~bash
cd ~/nobackup/cmsdas_2025_gen/
cp /eos/uscms/store/user/cmsdas/2025/short_exercises/generators/LHEReader.py .
~~~
{: .source}

#### Convert LHE output to root format

> ## Make sure the Python virtual environment is deactivated
> If you are still using the virtual environment, you need to unset it in order to not interfere with the CMSSW environment.
> {: .source}
{: .callout}
~~~bash
cd CMSSW_12_4_8/src; cmsenv; cd -
gzip -d MG5_aMC_v3_5_2/wplustest_4f_LO/Events/run_01/unweighted_events.lhe.gz
python3.9 LHEReader.py --input MG5_aMC_v3_5_2/wplustest_4f_LO/Events/run_01/unweighted_events.lhe --output standalone.root
python3.9 LHEReader.py --input genproductions_mg352/bin/MadGraph5_aMCatNLO/work/cmsgrid_final.lhe --output cmsgrid.root
~~~
{: .source}

Feel free to experiment here and plot various quantities. What are the shapes of the lepton pT distributions? What is the shape of the pT distribution of the W system? Are these shapes physical?

<!-- The most straightforward tool: MadAnalysis (MA).
MA is a tool designed to be used by theorists to analyze parton-level LHE files, particle-level HEPMC files or even events with DELPHES detector simulation.
We can install MA directly from the MG5 console.

~~~bash
cd ~/nobackup/cmsdas_2025_gen/MG5_aMC_v3_5_2/
./bin/mg5_aMC
~~~
{: .source}

You can then install MA from within MG
~~~
install MadAnalysis5 --madanalysis5_tarball=/eos/uscms/store/user/cmsdas/2025/short_exercises/generators/ma5_v1.9.13.tgz
~~~
{: .source}

> ## MadAnalysis mirror
> You might realize that we're installing MA from a local copy (`/eos/uscms/store/user/cmsdas/2025/short_exercises/generators/ma5_v1.9.13.tgz`).
> We've experienced some issues with using a [direct download](https://madanalysis.irmp.ucl.ac.be/raw-attachment/wiki/MA5SandBox/ma5_v1.9.13.tgz),
> therefore, we've prepared the local copy.
{: .callout}

After finishing the installation quit MG and run MadAnalysis
~~~bash
cd ../CMSSW_12_4_7/src;cmsenv;cd -
./HEPTools/madanalysis5/madanalysis5/bin/ma5
~~~
{: .source}
Select 2 cores for compiling (remember that you're on a shared computing node!) - this only needs to be done once.
LHE files can be imported as different datasets:
~~~
import wplustest_4f_LO/Events/run_01/unweighted_events.lhe as STANDALONE
import ../genproductions_mg265/bin/MadGraph5_aMCatNLO/work/cmsgrid_final.lhe as GRIDPACK
~~~
{: .source}

Now we can use simple syntax to define the distributions we would like to look at and start the analysis.
~~~
plot PT(mu+) 20 0 100
plot PT(mu-) 20 0 100
plot PT(l+) 20 0 100
plot PT(mu+ vm) 20 0 100
plot M(mu+ vm) 40 40 120
plot M(e+ ve) 40 40 120
plot M(ta+ vt) 40 40 120
plot M(l+ vl) 40 40 120

set main.stacking_method = superimpose
submit
~~~
{: .source}

As you can see from the terminal output, MadAnalysis translates the plotting commands into a C++ analyzer and compiles it. If you want to do some analysis steps that are not supported in the simple command-line syntax we used here, you can also write a C++ analyzer directly (MA calls this "expert mode"). This takes a little more time but gives you much greater flexibility.
The analysis output can be viewed as HTML. Since there is no web browser on cmslpc-sl7, let's copy ANALYSIS_0/Output/ to a local machine and open the HTML output:

firefox ANALYSIS_0/Output/HTML/MadAnalysis5job_0/index.html

What observations do you make? Are the two datasets consistent? What are the shapes of the lepton pT distributions? What is the shape of the pT distribution of the W system? Are these shapes physical?
Feel free to experiment here and plot other quantities you find interesting. You can refer to the [user manual](https://arxiv.org/pdf/1206.1599.pdf) to learn about the syntax. -->

{% include links.md %}

