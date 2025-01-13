---
title: "2 - Parton Shower Generator"
teaching: 10
exercises: 20
questions:
- "Why do we need to do parton showering?"
- "How are simulated samples created in CMS?"
objectives:
- "Perform parton shower with LHE file as an input"
- "Perform parton shower with gridpack as an input"
- "Analyze generator level information using NanoGEN files"
keypoints:
- "Pythia8 is the main tool used for parton showering in CMS"
- "Events are not physical if it did not go through parton shower"
- "Jet merging is a technique to avoid double countings of jet phase spaces in ME and PS calculations"
---

# Creating particle level samples from LHE files

As discussed earlier, LHE files itself are not enough to describe physical distributions.
In order to generate physics-wise sensible events, LHE files need to go through the parton shower.
Parton shower, in principle, is responsible for higher order corrections to the hard process.
Dominant contributions of such correction happen with collinear or soft emissions.
In CMS, one of the most widely used tool for parton shower is Pythia8 (however, do note that Pythia8 is a multipurpose generator that is able to calculate hard process for certain physics processes).
In this exercise, instead of compiling Pythia8 and running it in standalone mode as we did for MadGraph, we will take Pythia8 that is already compiled under CMSSW environment.

## (1) Running Pythia8 interface in CMSSW

> ## Make sure the Python virtual environment is deactivated
> If you are still using the virtual environment, you need to unset it in order to not interfere with the CMSSW environment.
> {: .source}
{: .callout}

Let's first check which release version of Pythia8 we will be using.

~~~bash
cd ~/nobackup/cmsdas_2025_gen/CMSSW_13_2_9/src
cmsenv
scram tool info pythia8
~~~
{: .source}

You can find out that we are now using Pythia8.309 version that is already compiled in `CMSSW_13_2_9`.
~~~
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Name : pythia8
Version : 309-3682aa03a0d969e762840638b179af29
++++++++++++++++++++

INCLUDE=/cvmfs/cms.cern.ch/el9_amd64_gcc11/external/pythia8/309-3682aa03a0d969e762840638b179af29/include
LIB=pythia8
LIBDIR=/cvmfs/cms.cern.ch/el9_amd64_gcc11/external/pythia8/309-3682aa03a0d969e762840638b179af29/lib
PYTHIA8DATA=/cvmfs/cms.cern.ch/el9_amd64_gcc11/external/pythia8/309-3682aa03a0d969e762840638b179af29/share/Pythia8/xmldoc
PYTHIA8_BASE=/cvmfs/cms.cern.ch/el9_amd64_gcc11/external/pythia8/309-3682aa03a0d969e762840638b179af29
ROOT_INCLUDE_PATH=/cvmfs/cms.cern.ch/el9_amd64_gcc11/external/pythia8/309-3682aa03a0d969e762840638b179af29/include
SYSTEM_INCLUDE+=1
USE=root_cxxdefaults cxxcompiler hepmc3 hepmc lhapdf
~~~
{: .output}

Now we will start building our parton shower fragment in our own directories in order to produce samples by ourselves.

~~~bash
mkdir -p Configuration/GenProduction/python/
~~~
{: .source}

Create a new file `Configuration/GenProduction/python/wplustest.py`:
~~~python
import FWCore.ParameterSet.Config as cms

import os

externalLHEProducer = cms.EDProducer('ExternalLHEProducer',
    args = cms.vstring(os.getenv("HOME")+ "/nobackup/cmsdas_2025_gen/genproductions_mg352/bin/MadGraph5_aMCatNLO/wplustest_4f_LO_el9_amd64_gcc11_CMSSW_13_2_9_tarball.tar.xz"),
    nEvents = cms.untracked.uint32(5000),
    numberOfParameters = cms.uint32(1),
    outputFile = cms.string('cmsgrid_final.lhe'),
    scriptName = cms.FileInPath('GeneratorInterface/LHEInterface/data/run_generic_tarball_cvmfs.sh'),
    generateConcurrently = cms.untracked.bool(False),
)

from Configuration.Generator.Pythia8CommonSettings_cfi import *
from Configuration.Generator.MCTunesRun3ECM13p6TeV.PythiaCP5Settings_cfi import *

generator = cms.EDFilter("Pythia8HadronizerFilter",
    PythiaParameters = cms.PSet(
        pythia8CommonSettingsBlock,
        pythia8CP5SettingsBlock,
        parameterSets = cms.vstring(
            'pythia8CommonSettings',
            'pythia8CP5Settings',
        )
    ),
    comEnergy = cms.double(13600),
    maxEventsToPrint = cms.untracked.int32(1),
    pythiaHepMCVerbosity = cms.untracked.bool(False),
    pythiaPylistVerbosity = cms.untracked.int32(1),
)
~~~
{: .source}

Let's compile.
~~~bash
scram b
~~~
{: .source}

`cmsDriver.py` executable makes the full configuration file based on the optional arguments it is given with (data tier, campaign, etc.) using the parton shower fragment that is built.
We will create NanoGEN files that are flat ntuples that resembles the NanoAOD data tier but only stored with generator-level information related branches.
It skips the SIM and RECO steps in the middle which makes it convenient to do generator-level studies.
For more information, take a look at [link](https://twiki.cern.ch/twiki/bin/viewauth/CMS/NanoGen).

~~~bash
cmsDriver.py Configuration/GenProduction/python/wplustest.py \
    --python_filename config.py \
    --eventcontent NANOAOD \
    --datatier NANOAOD \
    --fileout file:wplustest.root \
    --conditions auto:mc \
    --step LHE,GEN,NANOGEN \
    --no_exec \
    --mc \
    -n 100
~~~
{: .source}

You just created `config.py` that can be executed with `cmsRun` command.
Take a look at `config.py` with `less`, how it evolved from `Configuration/GenProduction/python/wplustest.py` through `cmsDriver.py`.
It will produce LHE files, run parton shower to make GEN samples, and then finally convert it to NanoGEN format in one go by doing below.
Note that we will only test with 100 events (`-n 100`) due to time constraints.

~~~bash
cmsRun config.py
~~~
{: .source}

LHE files are first produced using the gridpack we've just produced.
~~~
   ______________________________________     
         Running Generic Tarball/Gridpack     
   ______________________________________     
gridpack tarball path = /uscms/home/jlawless/nobackup/cmsdas_2025_gen/genproductions_mg352/bin/MadGraph5_aMCatNLO/wplustest_4f_LO_el9_amd64_gcc11_CMSSW_13_2_9_tarball.tar.xz
%MSG-MG5 number of events requested = 100
%MSG-MG5 random seed used for the run = 234567
%MSG-MG5 thread count requested = 1
%MSG-MG5 residual/optional arguments = 
%MSG-MG5 number of events requested = 100
%MSG-MG5 random seed used for the run = 234567
%MSG-MG5 number of cpus = 1
%MSG-MG5 SCRAM_ARCH version = el9_amd64_gcc11
%MSG-MG5 CMSSW version = CMSSW_13_2_9
WARNING: Developer's area is created for non-production architecture el9_amd64_gcc11. Production architecture for this release is el8_amd64_gcc11
Running MG5_aMC for the 1 time
produced_lhe  0 nevt  100 submitting_event  100  remaining_event  100
run.sh 100 2345670
Now generating 100 events with random seed 2345670 and granularity 1
~~~
{: .output}

Reweight with additional PDF sets given for possible systematic sources.

~~~
INFO: #***************************************************************************
#
# original cross-section: 30775.0
#     scale variation: +11.8% -12.7%
#     emission scale variation: + 0% - 0%
#     central scheme variation: +8.43e-09% -20.3%
# PDF variation: +0.918% -0.918%
#
#PDF NNPDF31_nnlo_as_0118_nf_4: 30776.1 +0.916% -0.916%
#PDF NNPDF30_nnlo_nf_4_pdfas: 29939.4 +1.81% -1.81%
#PDF NNPDF40_nnlo_nf_4_pdfas: 31022.5 +0.554% -0.554%
#PDF MSHT20nnlo_nf4: 30286.6 +1.2% -1.56%
#PDF PDF4LHC21_40_pdfas_nf4: 30529 +1.53% -1.53%
#PDF ABMP16_4_nnlo: 30385.1 +0.885% -0.885%
# dynamical scheme # 1 : 28597.7 +13.2% -14.3% # \sum ET
# dynamical scheme # 2 : 28599.7 +13.2% -14.3% # \sum\sqrt{m^2+pt^2}
# dynamical scheme # 3 : 24520.5 +16.6% -17.8% # 0.5 \sum\sqrt{m^2+pt^2}
# dynamical scheme # 4 : 30775 +11.8% -12.7% # \sqrt{\hat s}
# PDF 42930 : 30365.485849169454
#***************************************************************************
~~~
{: .output}

And then Pythia8 is launched with the LHE file created given as an input.
It first prints out the LHE information as we saw directly in the LHE file.

~~~
  --------  PYTHIA Event Listing  (hard process)  -----------------------------------------------------------------------------------

    no         id  name            status     mothers   daughters     colours      p_x        p_y        p_z         e          m
     0         90  (system)           -11     0     0     0     0     0     0      0.000      0.000      0.000  13000.000  13000.000
     1       2212  (p+)               -12     0     0     3     0     0     0      0.000      0.000   6500.000   6500.000      0.938
     2       2212  (p+)               -12     0     0     4     0     0     0      0.000      0.000  -6500.000   6500.000      0.938
     3         -1  (dbar)             -21     1     0     5     0     0   501     -0.000      0.000      0.771      0.771      0.000
     4          2  (u)                -21     2     0     5     0   501     0      0.000     -0.000  -2136.814   2136.814      0.000
     5         24  (W+)               -22     3     4     6     7     0     0      0.000      0.000  -2136.043   2137.585     81.187
     6        -13  mu+                 23     5     0     0     0     0     0     11.363     36.276  -1442.911   1443.412      0.106
     7         14  nu_mu               23     5     0     0     0     0     0    -11.363    -36.276   -693.132    694.173      0.000
                                   Charge sum:  1.000           Momentum sum:      0.000      0.000  -2136.043   2137.585     81.187

 --------  End PYTHIA Event Listing  -----------------------------------------------------------------------------------------------
~~~
{: .output}

Starts the parton shower on top of the given LHE event.
See how much more information gets printed out.
Remember that parton shower goes lower and lower from the hard process until certain energy threshold (`q -> q g -> q g g g -> q q q g g -> ...`).

~~~
  --------  PYTHIA Event Listing  (complete event)  ---------------------------------------------------------------------------------

    no         id  name            status     mothers   daughters     colours      p_x        p_y        p_z         e          m
     0         90  (system)           -11     0     0     0     0     0     0      0.000      0.000      0.000  13000.000  13000.000
     1       2212  (p+)               -12     0     0    90     0     0     0      0.000      0.000   6500.000   6500.000      0.938
     2       2212  (p+)               -12     0     0    91     0     0     0      0.000      0.000  -6500.000   6500.000      0.938
     3         -1  (dbar)             -21     6     0     5     0     0   501     -0.000      0.000      0.771      0.771      0.000
     4          2  (u)                -21     7     7     5     0   501     0      0.000     -0.000  -2136.814   2136.814      0.000
     5         24  (W+)               -22     3     4     8     8     0     0      0.000      0.000  -2136.043   2137.585     81.187
     6         21  (g)                -41    10     0     9     3   502   501      0.000      0.000      1.719      1.719      0.000
     7          2  (u)                -42    11    11     4     4   501     0     -0.000     -0.000  -2136.814   2136.814      0.000
     8         24  (W+)               -44     5     5    12    12     0     0    -19.744    -26.752  -1604.300   1606.697     81.187
     9          1  (d)                -43     6     0    13    13   502     0     19.744     26.752   -530.795    531.836      0.330
    10         -4  (cbar)             -41    18     0    14     6     0   501      0.000      0.000      2.947      2.947      0.000
    11          2  (u)                -42    19    19     7     7   501     0      0.000      0.000  -2136.814   2136.814      0.000
    12         24  (W+)               -44     8     8    20    20     0     0     -1.064    -19.028  -1221.827   1224.670     81.187
    13          1  (d)                -44     9     9    17    17   502     0     27.853     30.104   -681.041    682.274      0.330
    14         -4  (cbar)             -43    10     0    15    16     0   502    -26.789    -11.076   -231.000    232.816      1.500
    15         -4  (cbar)             -51    14     0    22    22     0   503    -19.016     -8.750   -120.727    122.537      1.500
    16         21  (g)                -51    14     0    23    23   503   502     -5.668     -0.052   -161.724    161.823      0.000
    17          1  (d)                -52    13    13    21    21   502     0     25.748     27.830   -629.590    630.730      0.330
    18         -4  (cbar)             -41    25     0    24    10     0   504      0.000      0.000      3.067      3.067      0.000
    19          2  (u)                -42    26    26    11    11   501     0      0.000      0.000  -2136.814   2136.814      0.000
    20         24  (W+)               -44    12    12    27    27     0     0     -0.639    -17.937  -1205.912   1208.775     81.187
    21          1  (d)                -44    17    17    28    28   502     0     25.919     28.268   -639.604    640.753      0.330
~~~
{: .output}

After 1 event information is printed out, 100 events get processed and finally reports the cross section.

~~~
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Overall cross-section summary
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Process		xsec_before [pb]		passed	nposw	nnegw	tried	nposw	nnegw 	xsec_match [pb]			accepted [%]	 event_eff [%]
0		3.078e+04 +/- 2.327e+02		100	100	0	100	100	0	3.078e+04 +/- 2.327e+02		100.0 +/- 0.0	100.0 +/- 0.0
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total		3.078e+04 +/- 2.327e+02		100	100	0	100	100	0	3.078e+04 +/- 2.327e+02		100.0 +/- 0.0	100.0 +/- 0.0
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Before matching: total cross section = 3.078e+04 +- 2.327e+02 pb
After matching: total cross section = 3.078e+04 +- 2.327e+02 pb
Matching efficiency = 1.0 +/- 0.0   [TO BE USED IN MCM]
Filter efficiency (taking into account weights)= (100) / (100) = 1.000e+00 +- 0.000e+00
Filter efficiency (event-level)= (100) / (100) = 1.000e+00 +- 0.000e+00    [TO BE USED IN MCM]

After filter: final cross section = 3.078e+04 +- 2.327e+02 pb
After filter: final fraction of events with negative weights = 0.000e+00 +- 0.000e+00
After filter: final equivalent lumi for 1M events (1/fb) = 3.249e-02 +- 2.478e-04

=============================================
~~~
{: .output}

> ## How did the cross section change after parton shower?
> 
> MadGraph reported `# original cross-section: 30775.0`, 30775pb.
> After running parton shower with Pythia8, same cross section 30780pb is kept.
> Parton shower adds more and more vertices, but why does the cross section remain unchanged?
> 
> > ## Solution
> > Parton shower is unitary.
> > Sum of probability to branch (e.g `q -> q g`) and not branch is 1.
> > Hence, the cross sections is determined by the lowest order input (hard process).
> > 
> {: .solution}
{: .challenge}

> ## Bonus: How did the distribution change?
{: .challenge}


## (2) Jet merging samples

Hard process calculation has advantage in modeling of hard jets and heavy particle decays while parton shower is great for describing collinear and soft emissions.
For more realistic and reliable physics modeling of hard jets, for example in W+jet events, MadGraph can be used as below.

~~~
generate p p > w+, w+ > ell+ vl @0
add process p p > w+ j, w+ > ell+ vl @1
~~~
{: .code}

With such syntaxes, MadGraph produces W+jet process with 0 and 1 hard jet in the event.
If this sample goes through parton shower, as some portion of events (dentoed with `@1`) readily involves hard jet, it would be better at describing W+jet process with hard jet.
However consider the event `@0` emitting QCD particles from initial state radiation that could possibly form a jet that is hard enough.
Such phase space inherently possesses a problem of double counting as "W+jet with hard jet" event could come from both `@0` and `@1`.
To mitigate such issues and remove double counting of phase space contributions, jet merging technique is used.
Jet merging is set up with an artificial cut threshold called jet merging scale.
This scale decides whether an event will be accepted or not from both `@0` and `@1`.
Finally, only accepted events from the two processes will be merged and form one sample.
Very roughly, jet merging scale can be thought as the momentum of a jet.
If a jet in the event is hard enough above the threshold, events from `@0` are rejected while only accepting from `@1`.
On the other hand, if a jet in the event is not too hard below the threshold, events from `@0` are only accepted while rejecting `@1`.

> ## How to produce gridpack
> 
> How to set the Madgraph (`run card`) and Pythia (`fragment`)?
> > ## Hint
> >
> > ~~~
> > #*********************************************************************
> > # Matching - Warning! ickkw > 1 is still beta
> > #*********************************************************************
> >  0        = ickkw            ! 0 no matching, 1 MLM, 2 CKKW matching
> > ~~~
> > {: .code}
> > This flag tells MadGraph that the LHE files we are going to produce will later be going through jet merging in > > order to avoid double countings.
> > 
> > ~~~
> > #*********************************************************************
> > # Jet measure cuts                                                   *
> > #*********************************************************************
> >  0   = xqcut   ! minimum kt jet measure between partons
> > ~~~
> > {: .code}
> > When jet merging is turned on, `xqcut` needs to be set which presample the events for efficient jet merging.
> > Remember that some portion of events will be later discarded and never going to be used.
> > So there is no point of producing events that involve jets with too low energy scale at this LHE level since these will likely be removed.
> > 
> > ~~~python
> > generator = cms.EDFilter("Pythia8HadronizerFilter",
> >     PythiaParameters = cms.PSet(
> >         pythia8CommonSettingsBlock,
> >         pythia8CP5SettingsBlock,
> >         processParameters = cms.vstring(
> >             'JetMatching:setMad = off',
> >             'JetMatching:scheme = 1',
> >             'JetMatching:merge = on',
> >             'JetMatching:jetAlgorithm = 2',
> >             'JetMatching:etaJetMax = 5.',
> >             'JetMatching:coneRadius = 1.',
> >             'JetMatching:slowJetPower = 1',
> >             'JetMatching:doShowerKt = off', 
> >             'JetMatching:qCut = 19.',
> >             'JetMatching:nQmatch = 4',
> >             'JetMatching:nJetMax = 1',
> >             'TimeShower:mMaxGamma = 4.0'
> >         ),
> >         parameterSets = cms.vstring(
> >             'pythia8CommonSettings',
> >             'pythia8CP5Settings',
> >             'processParameters',
> >         )
> >     ),
> > ~~~
> > {: .code}
> {: .solution}
{: .challenge}

> ## Bonus: What is the cross section?
{: .challenge}
