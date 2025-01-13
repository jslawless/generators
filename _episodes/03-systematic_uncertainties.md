---
title: "3 - Analysis and systematic uncertainties"
teaching: 10
exercises: 20
questions:
- "How can we use generator information for quick studies?"
- "What are systematic uncertainties coming from theory inputs?"
objectives:
- "Use NanoGEN for exploratory studies and to gain experience with generator related uncertainties (PDF choice, scale, strong coupling constant)"
- "Study some theory related systematic uncertainties"
keypoints:
- "PDF uncertainties can be estimated from a set of different per-event weights. The method depends on the type of PDF set that is used (hessian, MC replicas)"
- "Scale and alphaS variations are another source of uncertainty in the prediction of a simulated sample and can be used to estimate systematic uncertainties"
---

# Analysis of NanoGEN

In the following we will explore a bit the content of NanoGEN samples, and how they can be used for doing first studies for a potentially interesting physics analysis.
The NanoGEN sample we've previously created contains several trees.
You can open the file in root to explore the content, e.g.

~~~bash
cd ~/nobackup/cmsdas_2025_gen/CMSSW_13_2_9/src; cmsenv
root -l wplustest.root
~~~
{: .source}

Within root, you can use `.ls` to list the content.
If you're interested in the branches contained in a certain tree you can 
~~~bash
.ls
Events->GetListOfBranches()->ls()
~~~
{: .source}

The NanoGEN sample contains, amongst others, collections of all the generated particles (GenPart), jets with different cone size parameters (GenJet, GenJetAK8), generated missing transverse momentum (GenMET) and generated leptons (GenDressedLepton).
In the exercise we will be using GenMET and GenDressedLepton to calculate the transverse mass of the lepton+MET system, MT, which is often used in analysis with leptonically decaying W bosons.

## Where do systematic uncertainties come from?

Several choices of parameters and settings in the event generators have an impact on the predicted overall event yield as well as the acceptance and object kinematics.
The most prominent examples are the choice of the parton distribution function (PDF), the chosen values of the renormalization and factorization scales and the value of the strong coupling constant.

## Estimating systematic uncertainties

Several different PDF sets are usually stored in samples that are centrally produced in CMS.
In order to save space, only one PDF set is stored in NanoAOD and NanoGEN.
For the UL campaign, the [NNPDF3.1](https://nnpdf.mi.infn.it) NNLO PDF set is used ([LHAPDF](https://lhapdf.hepforge.org/pdfsets.html)), which contains 103 per-event weights.
The stored weights called `LHEPdfWeight` are ratios w.r.t. the central value, therefore, the weight with index 0 should usually be 1 (if not it means that different PDFs were used for the central ME and the computed variations and extra care needs to be taken).
Indices 1-100 are different, linear independent PDF sets that can be used to estimate systematic uncertainties, while indices 101 and 102 correspond to the up and down variations of the strong coupling constant.

> ## Hessian and MC replicas
> Two different approaches for PDF sets exist: MC replicas and hessian eigenvectors.
> While the hessian sets used in CMS in (most) UL samples allows for estimation of the total uncertainty by using the squared sum of the individual variations,
> the situation is more ambigous for MC replicas.
{: .callout}

Similar sets of variations are available for the renormalization and factorization scales, `LHEScaleWeight`.
You can find the details about the variations in the branch documentation within the root file (see above on how the get the list of branches in root).


# Optional (MadSpin and BSM UFO model)

## Using MadSpin

Why is MadSpin in any case useful?
The answer lies in NLO calculations in QCD or loop-induced processes.
Let's launch MadGraph prompt shell again.
~~~bash
cd ~/nobackup/cmsdas_2025_gen/MG5_aMC_v3_5_2/
./bin/mg5_aMC
~~~
{: .source}

Now try making another simple example that is top pair production.
~~~
import model sm
generate p p > t t~ [QCD]
~~~
{: .output}
It would be not so difficult to realize `[QCD]` has been added in the process definition.
This is a flag which tells MadGraph that you wish to do the calculations at NLO in QCD.

Before going further, try concatenating top decays into a W boson and a b quark similar to what we did for `Z -> ee` example.
~~~
generate p p > t t~, t > w+ b [QCD]
generate p p > t t~ [QCD], t > w+ b
exit
~~~
{: .output}

You will find neither of these working and instead MadGraph complains with an error log saying `str : Decay processes cannot be perturbed`.
So it means that physics processes with decays of particles are are not possible for NLO calculations.
This is where MadSpin becomes necessary, for such cases where resonant particle cannot be decayed can be decayed using MadSpin.
Now lets get back to the working example to see how it works.

~~~
import model sm
generate p p > t t~ [QCD]
output TopPair
launch
shower = PYTHIA8
4
0
~~~
{: .output}

Two lines are noticably added, `shower = PYTHIA8` and `4` (which can be replaced with `madspin = ON`).
We are again not going to do the parton shower here.
This is because depending on which parton shower generator one chooses later, "counter term" calculation differs which accounts as negatively weighted events.

> ## Negative weighted events
> We won't cover what it is in the tutorial but important things to remember are that
> 1. Some portion of the events are negatively weighted so one needs to be careful with the normalization.
> 2. LHE files at NLO are even more unphysical than LHE files at LO before parton shower.
{: .challenge}

Press `tab` to turn off timer.
MadGraph again asks if you would like to edit the cards now including `madspin_card.dat`.
~~~
/------------------------------------------------------------\
|  1. param   : param_card.dat                               |
|  2. run     : run_card.dat                                 |
|  3. madspin : madspin_card.dat                             |
\------------------------------------------------------------/
~~~
{: .output}

If you take a look at the `run_card.dat`, you might notice that the template for it is quite different from when we did DY at LO.
Template for NLO is shown in (link)[https://github.com/cms-PdmV/GridpackFiles/blob/master/Campaigns/Run3Summer22/MadGraph5_aMCatNLO/Templates/NLO_run_card.dat] and for LO is shown in (link)[https://github.com/cms-PdmV/GridpackFiles/blob/master/Campaigns/Run3Summer22/MadGraph5_aMCatNLO/Templates/LO_run_card.dat].
Although MadGraph shares the same user interface, LO and NLO calculations run on totally different codes in the backend.
So NLO type `run_card.dat` does not work for LO calculations and vice versa.

Now take a look at `madspin_card.dat` by pressing `3`.

~~~
# specify the decay for the final state particles
decay t > w+ b, w+ > all all
decay t~ > w- b~, w- > all all
decay w+ > all all
decay w- > all all
decay z > all all
~~~
{: .output}

This card lets you define how you want your resonant particles to decay.
For example, if you do :
~~~
decay t > w+ b, w+ > e+ ve
decay t~ > w- b~, w- > mu- vm~
~~~
{: .output}

This forces top to decay into positron and antitop to decay into muon.
Remove unnecessary decay definitions and add these two lines to make a top pair sample that ends up giving you positron and a muon.
Before moving on, do `set run_card nevents 50` to save time, producing only 50 events.

You will see inclusive top pair production cross section being computed which includes all possible decays for the top quark.
~~~
   --------------------------------------------------------------
      Summary:
      Process p p > t t~ [QCD]
      Run at p-p collider (6500.0 + 6500.0 GeV)
      Number of events generated: 50
      Total cross section: 6.847e+02 +- 4.3e+00 pb
   --------------------------------------------------------------
~~~
{: .output}

And then you will see MadSpin doing its job, decaying the top quarks to desired channels.
~~~
************************************************************
*                                                          *
*           W E L C O M E  to  M A D S P I N               *
*                                                          *
************************************************************

...

INFO: decay channels for t : ( width = 1.4915 GeV ) 
INFO:        BR                 d1  d2 
INFO:    1.000000e+00            b  w+  
INFO:    
INFO:    
INFO: decay channels for w+ : ( width = 2.04793 GeV ) 
INFO:        BR                 d1  d2 
INFO:    3.333610e-01            d~  u  
INFO:    3.333610e-01            s~  c  
INFO:    1.111195e-01            e+  ve  
INFO:    1.111195e-01            mu+  vm  
INFO:    1.110390e-01            ta+  vt  
INFO:    
INFO:    
INFO: decay channels for t~ : ( width = 1.4915 GeV ) 
INFO:        BR                 d1  d2 
INFO:    1.000000e+00            b~  w-  
INFO:    
INFO:    
INFO: decay channels for w- : ( width = 2.04793 GeV ) 
INFO:        BR                 d1  d2 
INFO:    3.333610e-01            d  u~  
INFO:    3.333610e-01            s  c~  
INFO:    1.111195e-01            e-  ve~  
INFO:    1.111195e-01            mu-  vm~  
INFO:    1.110390e-01            ta-  vt~

...

INFO:    Estimating the maximum weight     
INFO:    *****************************     
INFO:      Probing the first 75 events 
INFO:      with 400 phase space points 
INFO:    
INFO: Event 1/75 :  0.068s   
INFO: Event 6/75 :  0.63s   
INFO: Event 11/75 :  1.2s   
INFO: Event 16/75 :  1.8s   
INFO: Event 21/75 :  2.1s   
INFO: Event 26/75 :  3s   
INFO: Event 31/75 :  3.8s   
INFO: Event 36/75 :  4.6s   
INFO: Event 41/75 :  5.7s   
INFO: Event 46/75 :  6.5s   
~~~
{: .output}

> ## What is the cross section?
>
> Inclusive cross section was reported to be 684.7pb as we saw above.
> When considering the decay channels (`e+` and `mu-` final states), what is the proper cross section?
> What are the branching ratios for `w+ > e+ ve` and `w- > mu- vm~`?
>
> > ## Solution
> >
> > 8.5pb (from 684.7 x 11% x 11%)
> {: .solution}
> ## How can we make a sample that yields `mu+`, `vm`, and this time, two quark jets (hadronically decaying `w-`)
>
> > ## Solution
> > ~~~
> > decay t > w+ b, w+ > mu+ vm
> > decay t~ > w- b, w- > j j
> > ~~~
> > {: .output}
> {: .solution}
> 
{: .challenge}

# Interfacing BSM UFO model files

Let's take a look at how BSM samples for search type of analyses gets produced.
We will pick one simple example, a hypothetical heavy gauge boson that is called W' particle.

~~~
import model WEff_UFO
display particles
generate p p > wp+, wp+ > e+ ve
add process p p > wp-, wp- > e- ve~
output WprimeToENu
~~~
{: .output}


> ## How can we make the syntax simpler using particle containers?
>
> How can we write `generate p p > wp+, wp+ > e+ ve` and `add process p p > wp-, wp- > e- ve~` in a simpler way?
>
> > ## Solution
> > ~~~
> > define wprime = wp+ wp-
> > define leptons = e+ e- ve ve~
> > generate p p > wprime, wprime > leptons leptons
> > ~~~
> > {: .output}
> > This will find all possible Feynman diagrams with given particle combinations.
> > 
> {: .solution}
> 
{: .challenge}

As we are missing right-handed interactions for W bosons in the SM, a lot of BSM scenarios predict the W' boson that is heavier in mass (thus, we couldn't find it yet) but possesses the ability to interact with right-handed couplings.
As we do not know how large the particle's mass is, we test many different scenarios (BSM parameters), for example, different masses, decay channels, coupling strengths.
We will now see how such BSM parameters can be set in MadGraph.

~~~
launch
0
~~~
{: .output}

And press `tab` to turn off the timer.

Take a look at the parameter card by hitting `1`.

Now you will see there is a clear difference in the parameter settings when compared to the `sm` model file we've been using.
Here, we will only be focusing on the mass of W' `MWp` and the right-handed coupling strength `kR`.
In addition, you will also need to keep in mind that widths of the W' `wwp` should be changing based on how you choose your BSM parameters.

~~~
###################################
## INFORMATION FOR MASS
###################################
Block mass
    1 5.040000e-03 # MD
    2 2.550000e-03 # MU
    3 1.010000e-01 # MS
    4 1.270000e+00 # MC
    5 4.700000e+00 # MB
    6 1.720000e+02 # MT
   11 5.110000e-04 # Me
   13 1.056600e-01 # MMU
   15 1.777000e+00 # MTA
   23 9.118760e+01 # MZ
   25 1.250000e+02 # MH
   34 1.000000e+03 # MWp

...

###################################
## INFORMATION FOR WPCOUP
###################################
Block wpcoup
    1 0.000000e+00 # kL
    2 1.000000e+00 # kR

...

###################################
## INFORMATION FOR DECAY
###################################
DECAY   6 1.508336e+00 # WT
DECAY  23 2.495200e+00 # WZ
DECAY  24 2.085000e+00 # WW
DECAY  25 4.070000e-03 # WH
DECAY  34 1.000000e+01 # WWp
~~~
{: .output}

You can see that the mass of W' is now set to 1000GeV, right-handed coupling strength is set to 1.0, and the width of W' is given with 10GeV.
You can change the BSM parameters, maybe mass to 2000GeV and coupling strength to 0.1 by doing below.

~~~
set param_card mwp 2000
set param_card kr 0.1
~~~
{: .output}

However, if you again take a look at the parameter card, the width of W' `wwp` is kept same.
You can interactively see how the width value gets computed by doing `compute_widths wp+`.
Check the parameter card again, and you would see that width has changed and also tells you the branching ratios to different channels.

~~~
#      PDG        Width
DECAY  34   6.672601e-01
#  BR             NDA  ID1    ID2   ...
   2.506959e-01   2    2  -1 # 0.1672793598579319
   2.479126e-01   2    6  -5 # 0.16542221070326676
   2.379169e-01   2    4  -3 # 0.15875247762227632
   8.356529e-02   2    12  -11 # 0.05575978661997229
   8.356529e-02   2    14  -13 # 0.05575978638653866
   8.356519e-02   2    16  -15 # 0.05575972059211703
   1.277883e-02   2    2  -3 # 0.008526805639865994
~~~
{: .output}

Instead of doing interactive width computation, you can do `set param_card wwp auto`.
Then instead of first computing the widths, MadGraph will calculate the widths on-the-fly while generating events (but results will be identical).

Proceed by hitting `0` and see how much cross section it gives you when hypothetically the W' boson exists and decays to the electron channel, assuming mass 2000GeV with right handed coupling 0.1.

~~~
  === Results Summary for run: run_01 tag: tag_1 ===

     Cross-section :   0.001016 +- 1.447e-06 pb
     Nb of events :  10000
~~~
{: .output}

> ## How can we check the cross section when mass is 2000GeV with right handed coupling 1.0?
>
>
> > ## Solution
> > Repeat the exercise above but this time 
> > ~~~
> > set param_card mwp 2000
> > set param_card kr 1.0
> > ~~~
> > {: .output}
> > And most importantly, do not forget to compute the width by adding :
> > ~~~
> > set param_card wwp auto
> > ~~~
> > {: .output}
> >
> > Then you will get the following result.
> > ~~~
> >   === Results Summary for run: run_01 tag: tag_1 ===
> >
> >     Cross-section :   0.1045 +- 0.0001681 pb
> >     Nb of events :  10000
> > ~~~
> > {: .output}
> > 
> {: .solution}
> ## How much did the cross section increase compared to the scenario when mass is 2000GeV with right handed coupling 0.1?
>
> How many interactions did the W' boson get involved in?
> > ## Solution
> > One vertex when producing it, another vertex when it decays to electron channel.
> > Thus two interactions (1./0.1) = 10 gets squared and thus result in 100 times larger cross section.
> >
> {: .solution}
{: .challenge}
