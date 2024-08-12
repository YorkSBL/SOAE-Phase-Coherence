


2024.07.15

==========
No Noise

o ran vdModelP5.m as per params below --> coherenceD1.mat
NOTE: a key change was P.lengthT= 330*4; 
o I then obtained the total integrated summed waveform via
> wfVD=sum(real(data.Z(:,:)),2)'
o Looking at the waveform (via visualizeVDP4.m), I then took what appears to be the steady-state bit via
> wfVDss=wfVD(21000:end)
o Finally saving that to a txt file as
> dlmwrite('coherenceD1summed.txt',wfVDss')

==========
With Noise
o re-running, but now with P.extT= 2; (i.e., 2-noise (uniform to all bundles))
--> MUCH slower (like by x100?; took ~6 hrs)
--> save as coherenceD2.mat 
o saved steady-state bit similarly via visualizeVDP4.m
> wfVDss=wfVD(21000:end)
> dlmwrite('coherenceD2summed.txt',wfVDss')
==> Running EXplotSOAEwfP6.py, things look good: coherence drops off to zero outside tonotopic range for tdPC (much less so for nnPC), and coherence peaks where mags peak (and drops to zero between). So no surprises there: V&D08 model w/ noise behaves (mostly) as expected and similar to SOAE data 


% -----------------------------------------------------------------------

clear
global nR;   % dummy variable for ODE solver update counter
% ---------------------------------------
% User Parameters (P is a structure containing these values)
% NOTE: {##} indicates default 2016 ARO value
% +++
P.N = 100; % number of active oscillators (really P.N-1) [80] {100}
P.CFtype= 1;  % tonotopic map type: 0-linear, 1-log (determines w) [0] {1}
P.frange= [1 4.5];    % min/max freqs. for tonotopic map (determines w) [1 5] {[1 4.5]}
P.CFnoise= 0;   % add noise to CF distribution? (determines w) [0] {1}
P.CFnoiseFact= 2*10^(-2);    % factor for noisiness of CF distrib. [10^(-2)] {2*10^(-2)}
P.chunkify= 0; % boolean re making the tonotopic map "chunky" or not 0=no, 1=yes [0]
P.rows= 29; % if P.chunkify=1, approx. how many groups? (since it can be noisy) {29}
P.rowNoise= 0.2; % adds a bit of variability as to the integer # of oscillators per CF
% +++
% Nearest-neighbor coupling properties (passive)
P.dR_base= 0.15; % dR (dissipative coupling) base value {positive = dissipative} [0.15] {0.15}
P.dR_noiseF= 0.0;  % dR irregularity factor (% re base) [0; though try small values like 0.05] {0}
P.dI_base= -1.0; % d (reactive coupling) base value  [-1] {-1}
P.dI_noiseF= 0.05;  % d irregularity factor (% re base) [0;...] {0.05}
P.kap_base= 1;       % kap, coupling term [1] (a la Wit & van Dijk 2012) {1}
% +++
% "Active" properties for bundles
P.eFORM= 0;      % form of 'active' term (see comments above) [0] {0}
P.e_base= 1; %  e (individual oscillator damping) base value [1] (positive=active,negative=passive) {1}
P.e_noiseF= 0.05;  % e irregularity factor (% re base) [0;...] {0.05}
P.B_base= 1;    % B (strength of {cubic} nonlinearity) base value [1] {1}
P.B_noiseF= 0.05;  % B irregularity factor (% re base) [0;...] {0.05}
% +++
% Papilla parameters (P.alpha=P.beta=0 means no papilla coupling)
P.wP= 2;      % papilla CF [kHz] [2] {2}
P.eP= -1;    % papilla damping {positive=active,negative=passive} [-1] {-1}
% +++
% Papilla coupling properties (passive)
P.dRP= -0.15;     % (individual) viscous coupling factor between papilla and bundles {negative val= positive damping??} [?] {-0.15}
P.dIP= 1;     % (individual) elastic coupling factor between papilla and bundles [?] {1}
P.alpha= 1;   % (total) viscous coupling factor between papilla and bundles [0] {1}
P.beta= 1;    % (total) elastic coupling factor between papilla and bundles [0] {1}
% +++
% Initial conditions
P.ICnoise= 1;   % add noise to initial condition (IC) distribution? [1] {1}
P.ICbase= 1;  % amplitude factor for IC distrib. [1] {1}
P.ICnoiseFact= 1;  % amplitude factor for IC noisiness (reqs. P.ICnoise=1) [1] {1}
% +++
% External drive (i.e., non-autonomous term)
P.extT= 0;   % 0-none, 1-single tone, 2-noise (uniform to all bundles) [0] {0,1}
P.extD= 1;   % 0-drives papilla only, 1-drives papilla and all bundles [0] {1}
P.A = 1.2; % amplitude  (Note: 'dynamic range' of model is relatively small [try 0.1-5] {5.2,...}
P.fe = 3.4; % tone freq. (requires extT=1) {3.4,...}
% ----
% Integration properties
P.lengthT= 330*4;    % length ('seconds') to run integration [660] {a bit kludgy; stems from Wit et al. 2012?} {330}
P.stepSize = 1/128; % integration step-size (1/'seconds') [1/128??] {1/128}
% ----
% solver type: 0 - fixed-step RK4 (ode4), 1 - Matlab's ode45 (adaptive step-size)
P.solver= 1; % {1}
% +++
% "Freeze" the roughness?? (only bundles have roughness, currently via: P.w, P.e, P.dR, P.dI, P.B, P.kap)
% Note: USE CAUTION (params. other than those noted above are used as specified here, so ensure self-consist.)
% NOTE: ICs are not frozen, nor is any dynamic noise (i.e., Brownian drive term)
P.freeze= 0;
P.freezeFL= 'yy.mat';  % name of parameter file to load (reqs. P.freeze=1)
P.freezeFS= 'yy.mat';  % filename to save params too (reqs. P.freeze=1)
% ----
% Save data to file?
P.saveit = 1; % save data structure to file?? {1}
P.saveName = 'coherenceD1';  % filename to save to (req. P.saveit=1)