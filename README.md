# PaymentRouting
Algorithms for routing in payment channel networks 

Based on GTNA (https://github.com/BenjaminSchiller/GTNA), this simulator allows you to design and analyze routing protocols 
payment channel networks 

Select a test to run in class PaymentTest, which either run some key algorithms on a small test graph stored in the data folder 
or generates Barabasi Albert or ErdosRenyi graphs to run the algorithms on


The master branch is the basic simulator without concurrency, the concurrency branch adds concurrency for a future version of the paper. 

##Code structure 

The payment routing-related code can be found under src/paymentroute. There are six (sub-)packages:
* datasets: Classes for generating synthetic datasets with capacities, balances, and transactions 
* route: the main routing functionality (see below for more details)
* route.attack: the DoS enabled by linkability for splitting 
* route.fee: routing with fees (for [Merchant](https://arxiv.org/pdf/2012.10280.pdf)) 
* sourcerouting: source routing protocols such as Lightning (in progress)
* util: some helper classes 
 Only the first three are involved in the experiments for  [Splitting payments locally while routing interdimensionally](https://eprint.iacr.org/2020/555.pdf)
 
 The classes in route fall into categories:
 * Distance functions: DistanceFunction (abstract class), HopDistance, SpeedyMurmurs, SpeedyMurmursMulti (=Interdimensional) 
 * Splitting: PathSelection (abstract class), ClosestNeighbor (= no split), SplitClosest (=Split by Dist), SplitIfNecessary, RandomSplit, SplitIfTooHigh (not evaluated)
 * Governing of the routing process and computation of results: RoutePayment, PartialPath
 * Experiments: Evaluation, PaymentTests, SummarizeResults 
 
 ##Running Experiments

For reproducing the experiments, you can run settings in the class src/paymentroute/route/Evaluation.java. Here are the functions actually used in the paper:
* evalValTrees(): runs the experiments on the Lightning snapshot without concurrency and dynamics, Figure 3,4, and 6a in the paper. Results are written to the folder ./data/lightning-nopadding/
* topologyEval(): runs the comparison of different topologies, see Figure 6b in the paper. Results are written to the folder ./data/topology/
* locksEval(): runs the experiments for lower locktimes, see Figure 7 in the paper. Results are written to the folder ./data/locks-nopadding/
* attackEval(): runs the attack on linkability, see Figure 5 of the paper; results are written to ./data/attack-lightning/  
* dynamicEval(): runs the evaluation with adjusting balances, see Figure 8 of the paper; results are written to ./data/dyn-lightning/ 

All the above functions execute a large number of experiments for multiple parameter settings. If you want to test if the code runs and produces sensible results, please use the functions in the class PaymentTests, which provide results on small graphs for one specific setting. We suggest to first run the function 'runSimpleTest()', which will execute the four splitting algorithms from the paper (including random splitting) for HopDistance and InterdimensionalSpeedyMurmurs with 2 trees on a graph of 5 nodes. 

## Looking at the results 
The class SummarizingResults has a number of functions to retrieve concrete statistics from the results, see the respective comments in the code.

If you want to check the raw data: Each setting results in one result folder, the folder name reflects the parameters used to generate the network topology and transactions. Within these folder, there are subdirectories for each run and for each applied routing algorithm, a folder that summarizes the performance metrics over the runs.
Concretely, a result folder for one specific network is of the form READABLE_FILE_<GRAPHNAME>-nodecount (if the graph is read from a file) or of the form graphtype-nodecount--INIT_CAPACITIES-averageCapacity-capacityDistribution--TRANSACTIONS-averageTransactionValue-transactionDistribution-cutoff?-numberTransactions-timeValueAssociated?-onlyPossibleTransactions? (if the graph is generated by the simulation).
For instance, BARABASI_ALBERT-30-3--INIT_CAPACITIES-200.0-EXP--TRANSACTIONS-20.0-EXP-false-15-false-false indicates a Barabasi Albert graph of size 30 with average degree of about 6 (= 2*3), initial capacities with an average value of 200 that are exponentially distributed, transactions with average value 20, exponentially distributed without cutoff (e.g., there is no upper bound on the transaction value), 15 transactions, transactions are not given a concrete time value but are executed sequentially (time values come into play for concurrency), and we do not restrict the set of to those who are guaranteed to succeed. 
A subdirectory for a routing algorithm is of the form ROUTE_PAYMENT-splittingMethod-attempts-update?-distanceFunction-timelock. For instance,ROUTE_PAYMENT-CLOSEST_NEIGHBOR-1-false-HOP_DISTANCE-2147483647 indicates that no splitting is applied, routing is only attempted once, capacities are not dynamically adjusted, HopDistance is used as the distance function and there is no limit on the length of the routing path. ROUTE_PAYMENT-SPLIT_IFNECESSARY-1-true-SPEEDYMURMURS_MULTI_2-12 indicates that Split If Necessary is applied, routing is only attempted once, capacities are dynamically adjusted, Interdimensional SpeedyMurmurs with 2 trees is applied, and the maximal path length is 12. 
Within each of the routing algorithm subdirectionaries, there are files with results. The most important of these files is _singles.txt. The file contains the results for statistics that can be summarized in one value rather than a distribution (which are all the other files). The file contains lines of the form METRIC= and 9 values, which indicate the distribution over multiple runs (as a consequence, the _single file only has one value when in the subdirectionary for a specific run). The most important metrics are:
* ROUTE_PAYMENT_SUCCESS: fraction of successful payments
* ROUTE_PAYMENT_SUCCESS_DIRECT: fraction of payments successful in first attempt 
* ROUTE_PAYMENT_MES_AV: average overhead of a payment in terms of number of messages 
* ROUTE_PAYMENT_MES_AV_SUCC: average overhead of a payment in terms of number of messages when considering only successful payments 
* ROUTE_PAYMENT_HOPS_AV: average path length (longest path for split payments)
* ROUTE_PAYMENT_HOPS_AV_SUCC: average path length (longest path for split payments) when considering only successful payments 

The first 4 of the 9 values are average, median, minimum, maximum of the x runs. The fifth value is the standard deviation. The last two values jointly form the confidence interval (by default for 95\% but can be changed to 99\%) 




