
# Sudoku Solver Readme

## Approach Explanation
### *Initial Modelling*
Following some reading about constraint satisfaction problems (CSP) I decided to use a common but effective algorithm to achieve a solution. The first step was to generalise the problem : 
* $$X$$, the set of variables
* $D$, the set of domains (one for each variable)
* $C$, the set of constraints that specify allowable combinations of values

I concluded that $X$ would be each sqaure in the sudoku board, The sudoku consists of a $9$x$9$ grid so there are $81$ variables in each problem. The naming convention used was *row*-*column*. The *rows* had lettering $\{A,B,...,I\}$, whilst the columns where numbered $\{1,2,...,9\}$. For example the first square (top-left) is mapped to the variable $A1$. Furthermore the set of domains is simple to start with, all variables will have the same domain in a blank grid which is the set of numbers $\{1,2,...,9\}$. During our implementation of the algorithm the domain of each variable will have to be reduced, this is due to the fact that some variables may already have a value assigned when we load the grid and others have their domain reduced by the different constraints. The last set-up step was the set of constraints. There are three main types of constraints in a sudoku:
1. Each row cannot have a repeated number. *( $9$ constraints)*
2.  Each column cannot have a repeated number. *( $9$ constraints)*
3.  Each of the $9$, $3$x$3$ grids inside the sudoku cannot have a repeated number. *( $9$ constraints)*

We can clearly see all the constraints have a similar format, consisting of $9$ variables. There are a total of $27$ constraints, all variables in the constraint must take a different value. A sudoku will have a valid solution if we are able to return a set of values which satisfies the following, for all the constraints:

* >$alldiff(x_1,x_2,...,x_9)=\{d_1,d_2,...,d_9 | d_i∈D(x_i),d_j∈D(x_j),d_i≠d_j\}$

An example of each type of constraint we mentioned above are:
1. >$alldiff(A1,A2,...,A9)$ -*row*
2. >$alldiff(A1,B1,...,I1)$ -*column*
3. >$alldiff(A1,A2,A3,B1,B2,B3,C1,C2,C3)$ -*$3$x$3$ grid*

## *Algorithm Selection and Implementation*
Once I had constructed the basic data structures to have the sudoku loaded in and the initialise the set of variables along with their respective domain and construct the constraints, we had to be able to narrow down the set of possible values a variable can take. In other words we had to implement a constraint propagation algorithm. The constraint propagation algorithm to reduce the values a domain of each variable had was **Arc Consistency**. By definition the variables $x_i$ and $x_j$ are *arc consistent* if for every value in the current domain $D_i$ there is some value in the domain $D_j$ that satisfies the binary constraint in the arc $(x_i,x_j)$. Once this arc consistency has been done there were three possible outcomes :
* The CSP contains a variable $x_i$ which has an empty domain $(D_i=ø)$, if this occurs our algorithm can immediately return  a sudoku grid full of $-1$ as it will have no consistent solution.
* There is only one possible value for each variable which means our algorithm has reached a *goal state* and has finished. We will return the consistent solution achieved.
* We still have multiple possible values for variables. In this case we have to search for a solution, to do this we will use a **backtracking algorithm**.

### Backtracking Search
To keep things simple I started by testing a basic implementation of a backtracking algorithm. The strategy was to select the next unassigned variable, they were ordered by row and then column, $(A1$ had the highest priority going down to $I9$ with the lowest$)$. Once the variable had been selected just tried all values in the domain of this variable and try to extend it to a solution in the CSP. If it found one, we returned it otherwise, just backtrack to the state where we started and try the next value, if we had found no solution and tried every value we would conclude that there was no consistent solution.

**Improvements**

As we were solving a CSP we could take advantage of their factored representation and therefore use domain independent solutions to improve the backtracking. With this taken into account I implemented a **Minimum-remaining-values** (MRV) heuristic, this made the algorithm choose the unassigned variable which had the least possible values. This mean that it chooses the variable which is most likely to cause failure and hence reduce the amount of unnecessary searches. Arc consistency reduced the domains of variables before we began the search and MRV allowed us to improve efficiency when choosing which variable to start the search with. There was still a futher possibility of improvement by using inference during the search, this way we could infer domain reductions on new variables. When a variable $x_i$ had been assigned **forward checking** allowed the algorithm to establish arc consistency with every unassigned variable $x_j$ which is connected to $x_i$ by a constraint. This deleted any value from the domain $D_j$ which was inconsistent with value chosen for $x_i$. By using forward checking we found a more powerful way to compute the data neccesary for the MRV heuristic. 

## Other Ideas
The algorithm was able to solver all sudoku's in less than $0.5$ seconds with the algorithm used which involved the arc consistency to propagate the constraints and then a backtracking search using forward checking and the MRV. In this section, I will give a brief overview of some algortihms I decided not to implement for different reasons, however they could achieve more efficiency or solve harder problems. Implementing a **PC-2** algorithm to do the constraint propagation instead of **AC-3** means we could solve some harder sudoku's from without the need of searching although the computational cost will increase. This is because *path consistency* looks at triples of variables instead of pairs as in *arc consistency*.A more powerful algorithm in the interleaving search and inference is the **MAC** *(Mantaining arc consistency)*. This will detect more inconsistencies than *forward checking* as it does a similar process but it does not recursively propagate the constraints when a domain has been revised. 