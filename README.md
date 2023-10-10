# stable-matching
Implementation of a variation of the Gale-Shapley Stable Matching algorithm to find a top-half stable matching (if it exists).

This algorithm considers matching "hospitals" with "residents". A top-half stable matching is a matching in which every hospital 
and every resident is matched with someone in the top half of their ranking. This program takes in the preference lists of each 
hospital and resident, and determines whether a top-half stable matching exists. If so, it finds such a matching.

## Input/Output

The input to our program should be:

1. The first line has one integer number, 2 ≤ n ≤ 2, 000. Hospitals and residents are labeled with numbers 1, 2, . . . , n.
2. In each of the next 2n lines, we are providing the preference lists of the hospitals (with the first resident in the list
   being most preferred and the last resident being least preferred) followed by the preference lists of the residents.

The algorithm outputs data to stdout in up to n + 1 lines, where the first line of the output is Yes or No (depending on 
whether there exists a top-half stable matching); if the answer is Yes, the next n lines should the resident that each 
hospital got matched to.

## Algorithm and Analysis

### Algorithm

We will first redefine the concept of a "valid" match in this context, which would be a pair of a hospital and resident, (h,r) 
where h is in the top n/2 hospitals in r's preference list, and $r$ is in the top n/2 residents in $h$'s list. 

For our algorithm, we will start by changing the preference lists of the hospitals to only include "valid" matches, so check all 
of the hospital's first $n/2$ ranked residents, and check those residents' preference lists to see if that hospital is in the top 
half of their preferences as well. We will only consider these valid matches. 

Next, the hospitals will only propose to their valid matches, still in the relative ordering they originally were. Much like the 
original G-S Algorithm, if the resident that the hospital selects is not currently matched, they will temporarily accept that hospital's 
offer, and if they are already matched, they will go with the hospital that is higher in their preference list, and the hospital that the 
resident does not prefer will be free until someone else accepts their proposal. We determine whether there is a top half stable matching 
based off of whether this algorithm finds one. If, at any point, a hospital is left free and has exhausted all viable matches, then no 
top-half stable matching exists, and "No" is outputted. If at the termination of the algorithm all hospitals and residents are matched, 
a top half stable matching exists, and "Yes" is outputted.

### Runtime Analysis

Our algorithm is much like the original G-S algorithm, just with some preparation to be done before execution. 

We can use a 2D matrix to store the preferences of the residents to determine whether a resident is a viable match for a hospital in O(1) 
time. We can call this matrix M, such that M[i][j] gives the ranking of hospital j according to resident i's preference list. We have to 
do this for each of the $n$ hospitals, and for each of them, we have to check if n/2 residents are viable matches, which gives a total 
run-time of determining each hospital's viable matches of O(n^2). 

This same matrix can be used inside the body of our loop to compare how a resident ranks two hospitals in $O(1)$ time, which means each 
iteration of our loop can be executed in O(1) time.

We then execute our variation of the G-S algorithm using the updated preference lists. In each iteration of our while loop, one hospital 
makes a proposal to one resident. If we let P(t) be the set of all proposals made by iteration $t$ of our while loop, we know P(t+1) 
must be strictly greater than P(t). We also know that the maximum number of viable matches a hospital can have is n/2, meaning the maximum 
number of proposals a single hospital can make is n/2. If each of the n hospitals makes n/2 proposals, then we can say that the absolute 
maximum size of P(\*) would be O(n^2/2), or O(n^2). Since the maximum number of possible proposals is of order O(n^2) and $P(\*) increases 
with each iteration, $P(*)$ can only increase a maximum of O(n^2) times, meaning the maximum number of iterations of our loop would be 
O(n^2). Since we saw above that each iteration of our loop can be executed in O(1) time, we have a maximum of O(n^2) iterations that each 
take O(1) time, giving us a total run-time of O(n^2).

### Proof of Correctness

We want to show that if there exists a top half stable matching, then our algorithm will output "Yes", and if there exists no such top half 
stable matching, it will output "No".

We'll start by proving the direction that if there exists no top half stable matching, our algorithm will output "No" by proving the 
contra-positive: If our algorithm doesn't output "No" (i.e. it outputs "Yes"), then there must exist a top half stable matching. We will do 
this via contradiction: assume that there doesn't exist a top half stable matching when our algorithm says there is one, and show why that 
leads to a contradiction. If our algorithm terminates with "Yes", then it means that it has found a perfect matching that must only include 
top-half matches because those are the only matches our algorithm considers. So if it must be top-half but we assume it isn't top-half 
stable, then it must be the case that the matching we found is not stable. So if we call the matching our algorithm finds M, it must be the 
case that there is some hospital h, and resident r such that (h,r) is an instability in M. This means there must be pairs (h,r') and (h',r)
in M where h prefers r to r' and r prefers h to h'. Since h prefers r to r' but is matched with r' in M, r' must be a top-half viable match 
for h, and thus r must be in the top half of h's preference list. Since r prefers h to h' but is matched with h' in M, then h' must be in 
the top half of r's preference list, and then so must h. Since h is in the top half of r's preference list and r is in the top half of h's 
preference list, (h,r) is a viable top-half match, and since h prefers r to r', h must have proposed to r before proposing to r'. However, 
since r' ends up with h' and we can observe that residents matches only get better over time, it must be that r prefers h' to h, which 
contradicts our initial assumption. So it must be the case that our matching M is stable. Thus, there must exist a top half stable match, 
so our initial assumption that there doesn't exist a top half stable match must be false, and therefore, if our algorithm outputs "Yes", 
there must exist a top half stable matching. 

We will now prove the opposite direction: If there exists a top half stable matching, our algorithm will output "Yes". We will once again 
prove this by proving the contra-positive, which is if our algorithm outputs "No," there cannot exist a top-half stable match, which we will 
once again do by contradiction. We'll start by assuming our algorithm fails to find a stable top-half matching when one exists, call it M, 
and we'll prove that this assumption leads to a contradiction. If our algorithm outputs "No" then it must be the case that there was at least 
one hospital, h, that proposed to all "viable" matches but was rejected by all of them. 

Since M is top-half stable, it must be the case that each hospital has a top-half stable match in $M$. We'll consider the first hospital, h, 
which is rejected by their match in M, call this resident r. We can observe that it remains true that residents remain matched from the point 
they are proposed to until the end of the execution of the algorithm, and that their matches continue to only get better for them, so it must 
be the case that in our execution of the algorithm, r ends up with some other hospital, call it h', which r prefers to h. 

But then who does $h'$ end up with in $M$? We'll call this resident r. Since h was the first hospital to be rejected by their match in M, h' 
must not have yet proposed to r at the point where r rejects h in favor of h'. Since the hospitals propose in decreasing order of their 
preferences, that means that h' must prefer r to r'. And since r rejected h in favor of h', it must be the case that r prefers h' to h. 
However, this means that (h',r) is an instability in M as h' prefers r to its current matching, r', and r prefers h' to its current matching, 
h. So our initial assumption that M was stable must be incorrect, so no top-half stable matching can exist. Therefore we can conclude that 
if our algorithm outputs "No", then there cannot exist a top-half stable matching.

We have proved both that if our algorithm outputs "Yes" there must exist a top-half stable matching, and if our algorithm outputs "No" there 
cannot exists a top-half stable matching. This allows us to conclude that our algorithm will output "Yes" if and only if there exists a 
top-half stable matching, and thus our algorithm must be correct.
