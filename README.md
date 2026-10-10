# Simulation Usage Instructions (Practical Version)
Aimed at readers who have already run the code once and want to modify parameters themselves. This document answers three questions: How to run? What parameters can be modified? What data is output? Finally, it details the definitions of the Table rule tables.
# Table of Contents
1.	Running Methods
2.	Overview of Runnable Programs
3.	Usage Instructions for Each Program (One by One)
4.	Quick Reference Table for Modifiable Parameters (By File)
5.	Interpretation of Output Data
6.	Detailed Definitions of Table Rule Tables (Key Focus)
7.	Recommended Experimental Design
# 1. Running Methods
##Environment
-JDK 8 + Maven (or any IDE supporting Java 8)
-Dependency: commons-math3 (already declared in pom.xml)
## Method 1: IDE (Recommended, Easiest)
-Open the E:\fuzzyblockchain directory using IntelliJ IDEA (recognized as a Maven project), right-click the main method of the target class → Run.
## Method 2: Maven Command Line
-Bash
cd E:\fuzzyblockchain

Step 1. Compile + export required dependency classpath for running
mvn compile dependency:build-classpath -Dmdep.outputFile=cp.txt

Step 2. Run any class with a main method (just replace the class name below)
java -cp "target/classes;$(cat cp.txt)" com.chentinghao.Fuzzy.FuzzyTrustExperiment


⚠️ Note: Classes such as IntuitionisticFuzzyTrustSystem, FuzzyTrustChanges, and PDTMTrustModel do not have a main method; they are "service classes" and cannot be run directly. To view the output of fuzzy inference, run FuzzyTrustExperiment (it calls the inference logic).
#2. Overview of Runnable Programs
All programs with a main method:
Class Name (After com.chentinghao.)	One-Sentence Function	Runtime	Key Modifiable Parameters
Fuzzy.FuzzyTrustExperiment	Parameter scan experiment: view the "Malicious↑ → Trust↓" curve	Second-level ⚡	step
Fuzzy.VotingConsensusSystem	Basic voting consensus (selects Top-8 trusted nodes to vote)	~1 minute	rounds, number of nodes, threshold
Fuzzy.VotingDefendConsensus	Adds malicious nodes + forged/legitimate data detection statistics	~1 minute	rounds, malicious ratio, threshold
Fuzzy.VotingDefendDynmicConsensus	Dynamic version: malicious probability dynamically changes with trust score	~1 minute	rounds, baseMaliciousRatio
Fuzzy.PoisonTrust	Poisson + Sigmoid trust model demonstration	Second-level ⚡	number of nodes, lambda
Fuzzy.RandomFactorGenerator	Tool demonstration for generating correlated random factors	Second-level ⚡	correlation, Beta parameters
FastBFT.FastBFTSystem	Three-phase BFT consensus simulation (100 nodes)	~1 minute	totalNodes, consensusNodes, rounds, threshold
PBFT.PBFTConsensusSimulation	PBFT timing simulation	Second-level ⚡	NUM_ITERATIONS
Ratf.RaftConsensusSimulation	Raft timing simulation	Second-level ⚡	NUM_ITERATIONS
DPoS.DPoSConsensusSimulation	DPoS timing simulation	Second-level ⚡	NUM_NODES, NUM_ITERATIONS
HotStuff.HotStuffSimulationWithDelay	HotStuff (with delay) simulation	Second-level	See parameters inside the file
# 3. Usage Instructions for Each Program (One by One)
## 3.1 Fuzzy.FuzzyTrustExperiment — The Quickest Introductory Experiment
Function: Scans two curves ("malicious behavior increasing from 0 to 1" and "trust behavior decreasing from 1 to 0") to observe how the trust score changes.
Modifiable Parameters (FuzzyTrustExperiment.java):
Location	Parameter	Default	Function
L8	step	0.1	Scanning step size; smaller means denser curves
L20-22	Spatiotemporal distance, trajectory overlap degree	0.1, 0.9	Fixed values for these two factors during the experiment
L25	Called FuzzyTrustChanges.calculateTrustScore(...)	—	Switch to version A to compare two inferences
Output:
Experiment 1: Malicious behavior increases, trust behavior is fixed at 1
Malicious behavior value: 0.0, Trust value: 0.72
Malicious behavior value: 0.1, Trust value: 0.66
... (Higher malice, lower trust)
Experiment 2: Trust behavior decreases, malicious behavior is fixed at 0
Trust behavior value: 1.0, Trust value: 0.72
... (Lower trust, lower trust score)

## 3.2 Fuzzy.VotingConsensusSystem — How Trust Scores Are Used for Voting
Function: Generates 100 nodes per round → selects the 8 nodes with the highest trust scores → each node determines its voting probability based on "trust score + spatiotemporal overlap degree" → consensus is reached if the sum of trust values for affirmative votes is ≥ 60% of the total trust value. Counts the "number of correct consensus achievements".
Modifiable Parameters:
Location	Parameter	Default	Function
L19	runConsensus(500)	500 rounds	Number of rounds
L33	IntuitionisticFuzzyTrustSystem.getTrustScore()	Version A	Switch to PDTMTrustModel.generateTrustResults() to compare another trust model
L43-50	Spatiotemporal thresholds	0.33 / 0.66	Divides into "Far/Medium/Near"
L53-81	Voting probability table	See §5	Higher trust score and closer distance → higher probability of voting in favor
L117	generateNodes(100)	100 nodes	Total nodes per round
L120	numVoters = 8	8	Top-K participating voters
L165	Threshold coefficient	0.6	Consensus threshold (60% of total trust value)
Output (Per round + final statistics):

Round 3 voting:
  Node-5 votes: Trust value = 0.82, Spatiotemporal overlap degree = 0.71 (Near) -> In favor
  ...
Voting result: Total trust value = 5.20, Affirmative vote trust value = 4.10, Threshold = 3.12 -> Consensus reached
...
Number of correct consensus achievements: 487 times

## 3.3 Fuzzy.VotingDefendConsensus — Malicious Nodes + Detection Statistics (Research Core)
Function: Adds two things compared to the above: ① Some nodes are malicious nodes (malicious nodes intentionally invert their votes); ② Each round is randomly divided into "forged data rounds (30%) / legitimate data rounds (70%)", and the system's ability to correctly identify them is counted. Node trust values come from PDTMTrustModel (Bayesian smoothing model).
Modifiable Parameters:
Location	Parameter	Default	Function
L27	runConsensus(500)	500 rounds	Number of rounds
L37	PDTMTrustModel.generateTrustResults()	—	Source of node trust
L123	Math.random() < 0.3	30%	Proportion of forged data rounds
L129 / L133	maliciousRatio	0.2	Proportion of malicious nodes (modifiable in both round types)
L137	generateNodes(100, ...)	100	Number of nodes per round
L139	numVoters = 8	8	Top-K voting participants
L175	threshold = totalTrustValue * 0.6	60%	Consensus threshold
Output (Per round + 4 key statistical metrics):

Round 12 consensus:
  Node-3 votes: Trust value = 0.55, Spatiotemporal overlap degree = 0.48 (Medium), Malicious = No -> In favor
  Node-7 votes: Trust value = 0.61, Spatiotemporal overlap degree = 0.52 (Medium), Malicious = Yes -> Against   ← Malicious node inverted its vote
  Current round: Total trust value = 4.90, Affirmative trust value = 3.30, Threshold = 2.94 -> Consensus result: true, True result: false
...
Forged data detection rate: 82.00%    ← Proportion of correctly judged false in forged rounds
Legitimate data detection rate: 91.00%    ← Proportion of correctly judged true in legitimate rounds
Forged data false alarm rate: 18.00%    ← Treating forged data as legitimate
Legitimate data false alarm rate: 9.00%     ← Treating legitimate data as forged

These four rates are the most commonly used evaluation metrics in papers: The higher the detection rate, the better; the lower the false alarm rate, the better.
## 3.4 Fuzzy.VotingDefendDynmicConsensus — Dynamic Malicious Proportion
Function: Structured the same as 3.3, but the malicious probability is dynamic—the lower the trust score, the higher the probability of being a malicious node:
finalOnlyTrustScore Range	Malicious Probability = base × factor
< 0.3	base × 1.0 (High-trust nodes also mix in, hardest to defend against)
0.3 ~ 0.66	base × 0.6
≥ 0.66	base × 0.3 (High-trust nodes are rarely malicious)
Modifiable Parameters:
Location	Parameter	Default	Function
L27	runConsensus(500)	500	Number of rounds
L45	IntuitionisticFuzzyTrustSystem.getTrustScore()	Version A	Source of trust
L56-64	Segmented thresholds 0.3 / 0.66	—	Dynamic factor boundaries
L146	baseMaliciousRatio	0.3	Base malicious proportion
Output: Same as 3.3, ending with the same 4 rates.

## 3.5 Fuzzy.PoisonTrust — Poisson Trust Model Demonstration
Function: Demonstrates another trust model (Log-Poisson + Sigmoid). Generates 10 nodes and prints the trust score for each.
Modifiable Parameters: PoisonTrust.java L86 numberOfNodes=10; L28 lambda=1.0; L103 interaction count N(50,15).
Output:
Node 0: Trust Score = 0.61, Spatial-Temporal Value = 0.45           ...

## 3.6 Fuzzy.RandomFactorGenerator — Factor Generation Tool Demonstration
Function: Generates 10 pairs of "trajectory overlap/spatiotemporal distance" and prints the actual correlation coefficient.
Modifiable Parameters: RandomFactorGenerator.java L96-98 (in main): correlation=-0.4, similarityBetaParams={4,3}, distanceBetaParams={3,4}.
Output:
Similarity	Distance
0.7213		0.3056
...
Generated Pearson correlation coefficient: -0.4181
Generated Spearman correlation coefficient: -0.3924


## 3.7 FastBFT.FastBFTSystem — Pure BFT Consensus Simulation
Function: 100 nodes, 20 randomly selected per round, 1 leader chosen → PRE_PREPARE/PREPARE/COMMIT three-phase voting, consensus reached if affirmative votes ≥ 2/3. Simulates 20~120ms network delay and counts total execution time.
Modifiable Parameters (FastBFTSystem.java L13-16):
Parameter	Default	Function
totalNodes	100	Total number of nodes
consensusNodes	20	Number of participating nodes per round
totalRounds	100	Number of rounds
thresholdRatio	2/3	Consensus threshold ratio
L89 sleep(480)	480ms	Approximate time waiting for messages
L341 Delay range	20~120ms	Simulates network delay
Output:

=== Round 5 consensus ===
Selected nodes: Node-12, Node-45, ...
Leader Node-45 proposes: Proposal-Round-5
Voting result: Affirmative votes = 15, Threshold = 13.33 -> Consensus reached
...
Total time for 100 rounds of consensus: 48.52 seconds
Average consensus time per round: 0.49 seconds


## 3.8 PBFT / Raft / DPoS — Timing Comparison Simulation
All three are simple simulations that "simulate the time consumption of each stage and count the total time", used to compare the time magnitudes of different algorithms.
File	Main Parameters
PBFT/PBFTConsensusSimulation.java	L19 NUM_ITERATIONS=100
Ratf/RaftConsensusSimulation.java	L17 NUM_ITERATIONS=100
DPoS/DPoSConsensusSimulation.java	L21 NUM_NODES=10, L22 NUM_ITERATIONS=100

# 4. Quick Reference Table for Modifiable Parameters (By File)
Format: file:line number → parameter = default value (function)
Fuzzy Inference (Core)
File:Line	Parameter	Default	Function
IntuitionisticFuzzyTrustSystem.java:11	Random seed	123	Fixed = reproducible experiments; changed = different each time
:12	correlation	-0.6	Negative correlation strength between trajectory and distance
:13-15	betaParams / similarityBetaParams / distanceBetaParams	{3,4} / {4,3} / {3,4}	Factor distribution shapes
:53-80	Trapezoidal membership vertices	See code	Fuzzification gear width (narrower means more sensitive)
:99-102	Defuzzification domain	{0,.25,.5,.75,1} / {0,.3,.6,1}	Score mapping range
:227-233	Table 3 Rule Table	See §6	Behavior → Objective Trust $O$
:276-280	Table 5 Rule Table	See §6	Trajectory × Distance → Spatiotemporal Overlap $S$
:321-325	Table 6 Rule Table	See §6	$S \times O$ → Real-Time Trust $FR$
:360	Defuzzification weight	$\mu + \pi$	Hesitancy weight coefficient
FuzzyTrustChanges.java:67-98	Version B trapezoid	Non-uniform	More sensitive gear division
Voting Consensus
File:Line	Parameter	Default	Function
VotingConsensusSystem.java:19	rounds	500	Consensus rounds
:117	Total nodes	100	Number of nodes generated per round
:120	numVoters	8	Top-K voting participants
:165	Threshold coefficient	0.6	Consensus threshold
VotingDefendConsensus.java:123	Forged round proportion	0.3	Forged/legitimate data distribution
:129, 133	maliciousRatio	0.2	Proportion of malicious nodes
VotingDefendDynmicConsensus.java:146	baseMaliciousRatio	0.3	Dynamic malicious probability base
:56-64	Dynamic factor threshold	0.3/0.66	Malicious probability segmentation
BFT / Other Consensus
File:Line	Parameter	Default
FastBFTSystem.java:13-16	totalNodes / consensusNodes / totalRounds / thresholdRatio	100 / 20 / 100 / 2⁄3
PBFTConsensusSimulation.java:19	NUM_ITERATIONS	100
RaftConsensusSimulation.java:17	NUM_ITERATIONS	100
DPoS/DPoSConsensusSimulation.java:21-22	NUM_NODES / NUM_ITERATIONS	10 / 100

# 5. Interpretation of Output Data
Common Output Line Format Explanations
Membership Print (from IntuitionisticFuzzyTrustSystem):
Trust behavior membership: F1=(μ=0.00, ν=1.00, π=0.00) F2=(μ=0.50, ν=0.50, π=0.00) ...


→ $\mu/\nu/\pi$ for each gear, where $\mu + \nu + \pi = 1$.
Rule Activation Print:
Rule activation: Trust F5 & Malicious F1 -> O4 (μ=0.72, ν=0.25, π=0.00)


→ Indicates which combination triggered which rule and what the activation intensity is.
Voting Line (VotingDefend series):
Node-7 votes: Trust value = 0.61, Spatiotemporal overlap degree = 0.52 (Medium), Malicious = Yes -> Against

→ Malicious = Yes indicates that the node is a malicious node; note that it voted "Against" (inverted).
Consensus Determination Line:
Current round: Total trust value = 4.90, Affirmative trust value = 3.30, Threshold = 2.94 -> Consensus result: true, True result: false

→ Consensus result vs. True result inconsistent = False alarm.
Four Statistical Rates (End of VotingDefend series):
Metric	Meaning	Ideal Value
Forged data detection rate	Proportion of forged rounds judged as "non-consensus"	Higher the better (close to 100%)
Legitimate data detection rate	Proportion of legitimate rounds judged as "consensus"	Higher the better
Forged data false alarm rate	Forged rounds mistakenly judged as "consensus"	Lower the better
Legitimate data false alarm rate	Legitimate rounds mistakenly judged as "non-consensus"	Lower the better

# 6. Detailed Definitions of Table Rule Tables (Key Focus)
The three tables of fuzzy inference are the system's "expert knowledge". Modifying the table = modifying the system's decision-making logic.

## 6.1 General Concepts: Gears and Numerical Mapping
-Input side: F1~F5 = 5 fuzzy gears for trust/malicious behavior (Very Low → Very High); G1~G3 = 3 gears for trajectory overlap; S1~S3 = 3 gears for spatiotemporal distance.
-Output side: $O$ (Objective Trust) and $FR$ (Real-Time Trust) have 5 levels each; $S$ (Spatiotemporal Overlap) has 3 levels.
-The numbers in the code are "gear indices", where O[0] = lowest level and O[4] = highest level. During defuzzification, they are mapped to scores according to the domain:
Output	Gear Index → Score
O / FR	$0 \to 0.0$, $1 \to 0.25$, $2 \to 0.5$, $3 \to 0.75$, $4 \to 1.0$
S	$0 \to 0.0$, $1 \to 0.3$, $2 \to 0.6$ (The 4th element $1.0$ of the domain is unused)
## 6.2 Table 3: Trust Behavior $\times$ Malicious Behavior $\to$ Objective Trust Level $O$
Row = Trust behavior gear, Column = Malicious behavior gear, Intersection point = $O$ level (0~4)
Version A (IntuitionisticFuzzyTrustSystem.java:227):
          Malicious F1  Malicious F2  Malicious F3  Malicious F4  Malicious F5
Trust F1      2             1             1             0             0
Trust F2      2             2             1             1             0
Trust F3      3             2             2             1             1
Trust F4      4             3             2             2             1
Trust F5      4             4             3             2             2

Version B (FuzzyTrustChanges.java:245, stricter):
 Malicious F1  Malicious F2  Malicious F3  Malicious F4  Malicious F5
Trust F1      1             1             0             0             0
Trust F2      2             2             1             0             0
Trust F3      3             2             2             1             0
Trust F4      3             3             2             1             1
Trust F5      4             3             2             1             1

How to read: Top-left "Trust F5 & Malicious F1 $\to O = 4$" (honest and no malice $\to$ highest trust); bottom-right "Trust F1 & Malicious F5 $\to O = 0$" (almost dishonest and entirely malicious $\to$ lowest trust). Moving one column to the right (malicious increases by one gear) causes $O$ to drop by at least 1 level—this is the quantified implementation of "punishing malice".
How to modify: Reduce the number corresponding to a certain "high malicious gear" → harsher against malice. For example, change the Trust F5 row in Version A to {4, 3, 2, 1, 1} → even if trust behavior is high, encountering malice F4/F5 will only yield $O = 1$. ⚠️ Note that Version B's final score uses $O$ (see §7 old document), so modifying Table 3 directly takes effect for it; Version A goes through $FR$, which also requires looking at Table 6.

## 6.3 Table 5: Trajectory Overlap $\times$ Spatiotemporal Distance $\to$ Spatiotemporal Overlap Degree $S$
Row = Trajectory overlap gear $G$ ($G1$ Low ~ $G3$ High), Column = Spatiotemporal distance gear $S$ ($S1$ Near ~ $S3$ Far) (Both versions are identical, IntuitionisticFuzzyTrustSystem.java:276 / FuzzyTrustChanges.java:294):
Distance S1  Distance S2  Distance S3
G1 (Low overlap)    1            0            0
G2 (Medium overlap) 2            1            0
G3 (High overlap)   2            2            1


How to read: Higher trajectory overlap (moving down) and closer distance (moving left) $\to$ larger $S$ (higher spatiotemporal overlap degree). "G1 & S3" (no trajectory overlap and far distance) $\to S = 0$; "G3 & S1" (high trajectory overlap and close distance) $\to S = 2$.
Semantics: When a vehicle (node)'s trajectory highly overlaps with the normal trajectory and its distance to the event is close, it indicates that it is "present and behaving normally", resulting in higher spatiotemporal credibility.

## 6.4 Table 6: Spatiotemporal Overlap Degree $S \times$ Objective Trust $O \to$ Real-Time Trust Level $FR$
Row = $S$ (0~2), Column = $O$ (0~4)
Version A (IntuitionisticFuzzyTrustSystem.java:321):
O0  O1  O2  O3  O4
S0   0   1   1   2   3
S1   1   1   2   3   3
S2   1   2   3   4   4


Version B (FuzzyTrustChanges.java:339, stricter):
O0  O1  O2  O3  O4
S0   0   0   1   2   3
S1   0   1   2   3   3
S2   1   2   3   3   4

How to read: Both $S$ and $O$ are high $\to$ $FR$ is high (bottom-right $4 = \text{highest trust}$). If either $S$ or $O$ is low, $FR$ is suppressed—this reflects the embodiment of "spatiotemporal overlap degree" as a secondary trust verification check. Comparing the two versions: Version B is lower than Version A in row S0 (S0 & O1 drops from 1 to 0), indicating that it is more distrustful of situations with "low spatiotemporal overlap degree".

## 6.5 How Rules Are Activated and Combined (Code Logic)
For each pair of (input gear i, input gear j):
  If μ(i) ≤ 0 or μ(j) ≤ 0 → Skip (This rule is not triggered)
  Activation value: μ = min(μi, μj), ν = max(νi, νj), π = min(πi, πj)
  Falls into output gear k: O[k].μ = max(O[k].μ, activation μ)   ← Multiple rules in the same gear take the maximum
                             O[k].ν = min(O[k].ν, activation ν)
                             O[k].π = max(O[k].π, activation π)  (Only when activation μ < 0.9)

Aggregation Rule of Thumb: μ takes the minimum then the maximum, ν takes the maximum then the minimum, π takes the minimum then the maximum. The first half (min/max) is rule activation (bottleneck effect), and the second half (max/min) is rule combination.

## 6.6 Defuzzification: Converting Gears into Scores

$$\text{Final score} = \frac{\sum [(\mu + \pi) \times \text{score of this gear}]}{\sum (\mu + \pi)}$$
●	$\pi$ (hesitancy) and $\mu$ have equal weight → When information is insufficient, the score is pulled toward the representative value, making the system conservative.
●	To be more conservative: Change $(\mu + \pi)$ to something like $(\mu + 2\pi)$ to increase the weight of hesitancy.
All line numbers and default values in this document have been verified against the current code. It is recommended to back up the original text or copy the entire class before modifying parameters.
