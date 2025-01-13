---
title: Setup
---
# Set-up your working environment

> ## Mattermost (chat)
> There is a dedicated Mattermost team, called [CMSDAS@LPC2025](https://mattermost.web.cern.ch/cmsdaslpc2025/channels/town-square), setup to facilitate communication and discussions via live chat (which is also archived). The channel is hosted by the [CERN Mattermost instance](https://mattermost.web.cern.ch/).
> 
> If you have never used Mattermost at CERN, please know that you will need your CERN login credentials (SSO) and you will need to join the CMSDAS@LPC2025 channel in order to be able to see (or find using the search channels functionality) the channels setup for communications related to the school.
> 
> If you already have used Mattermost at CERN, please know that when you click direct links to channels (as you will find below) that are within the CMSDAS@LPC2025 channel, you **may** be redirected to the last Mattermost team you used. If this happens, remember to click the [signup link to join the CMSDAS@LPC2025 channel](https://mattermost.web.cern.ch/login?redirect_to=%2Fcmsdaslpc2025%2Fchannels%2Ftown-square) to switch to the correct team from which you should be able to see the individual channels. If that still doesn’t work, remove all cookies associated with cern.ch and restart your browser.
> 
> The [ShortExGenerators](https://mattermost.web.cern.ch/cmsdaslpc2025/channels/ShortExGenerators) will be available once you join or switch to the CMSDAS@LPC2025 channel!
> 
> Note that you can access Mattermost via browser or you can download the corresponding application running standalone on your laptop or smartphone. The laptop application does not work with CERN certificate login.
{: .prereq}

> ## Important
> **This exercise is designed to run only on the LPC clusters as copies of the scripts are present there.**
{: .callout}
## Set-up your working directory
Keep in mind your `<USERNAME>` and use it in the following instructions:
~~~bash
# login to the LPC cluster with DISPLAY set
ssh -Y <USERNAME>@cmslpc-el8.fnal.gov

# get to your no-backup working area
cd nobackup

# line to find out what shell you are in
echo $0

# create your working directory
mkdir cmsdas_2025_gen
~~~

<!-- The following commands are tested on the LPC cluster, using the bash shell.
You can run `echo "$SHELL"` to check that you are indeed running bash.
Add the following lines to your `~/.bash_profile` file
~~~bash
export CDGPATH=${HOME}/nobackup/cmsdas_2025_gen
source /cvmfs/cms.cern.ch/cmsset_default.sh
~~~
{: .source}
Run `bash -l` to restart bash, or alternatively `source ~/.bash_profile` to activate your change. You don't need to rerun this command next time you re-login to the LPC cluster. -->


## Downloading MadGraph

Downloading MadGraph5 v3.5.2 from a CMS mirror.
~~~bash
cd ~/nobackup/cmsdas_2025_gen
wget https://cms-project-generators.web.cern.ch/cms-project-generators/MG5_aMC_v3.5.2.tar.gz
tar xf MG5_aMC_v3.5.2.tar.gz
rm MG5_aMC_v3.5.2.tar.gz
~~~
{: .source}

## Running MadGraph
Please note that MG5_aMC_v3.5.2 requires Python 3.7 or higher. We will set up MadGraph with making virtual environments for Python 3.11 (available on the LPC clusters):
- Set up a Python 3 virtual environment
  ~~~bash
  python3.11 -m venv mg352
  ~~~
  {: .source}
- Activate the the Python 3 virtual environment and install `six` package
  ~~~bash
  source mg352/bin/activate
  python -m pip install six
  ~~~
  {: .source}
- Run MadGraph
  ~~~bash
  cd MG5_aMC_v3_5_2
  ./bin/mg5_aMC
  ~~~
  {: .source}

To quit the MadGraph interface use `exit` command.

Note: To deactivate the Python 3 virtual environment, type `deactivate`.

## Downloading CMS generator tool

Cloning the CMS generator repository:
~~~bash
cd ~/nobackup/cmsdas_2025_gen
git clone -b mg352 https://github.com/cms-sw/genproductions.git genproductions_mg352
~~~
{: .source}

You can check that the MadGraph steering cards of the example we will be using are actually there:
~~~bash
ls -rtl genproductions_mg352/bin/MadGraph5_aMCatNLO/cards/examples/wplustest_4f_LO/
~~~
{: .source}

~~~
total 20
-rw-r--r-- 1 enibigir us_cms   223 Jan  6 14:06 wplustest_4f_LO_proc_card.dat
-rw-r--r-- 1 enibigir us_cms 15999 Jan  6 14:06 wplustest_4f_LO_run_card.dat
~~~
{: .output}

Let’s copy the two files
~~~bash
cp genproductions_mg352/bin/MadGraph5_aMCatNLO/cards/examples/wplustest_4f_LO/*.dat MG5_aMC_v3_5_2/
~~~
{: .source}

Open `MG5_aMC_v3_5_2/wplustest_4f_LO_run_card.dat` using your favorite editor (vim, emacs,...)
and replace the following lines
~~~
250 = nevents ! Number of unweighted events requested
'lhapdf'    = pdlabel     ! PDF set
$DEFAULT_PDF_SETS = lhaid     ! if pdlabel=lhapdf, this is the lhapdf number
~~~
{: output}
with
~~~
10000 = nevents ! Number of unweighted events requested
nn23lo1    = pdlabel     ! PDF set                                     
230000     = lhaid       ! if pdlabel=lhapdf, this is the lhapdf number
~~~
{: output}

## Preparing CMSSW working area
~~~bash
cd ~/nobackup/cmsdas_2025_gen
cmsrel CMSSW_13_2_9
~~~
{: .source}

<!-- ## Preparing CMSSW working area
We start with pulling and installing the required CMS software (CMSSW).
We use two versions: the older 10.6.19 is used for sample generation,
while we will make use of the more up-to-date tools installed in 12.4.7 when we analyze the samples.
~~~bash
cd ~/nobackup/cmsdas_2025_gen
cmsrel CMSSW_12_4_8
~~~
{: .source} -->

<!-- Cloning extra scripts and config files
~~~bash
git clone https://github.com/danbarto/gen-cmsdas-2023.git
~~~
{: .source} -->


{% include links.md %}
