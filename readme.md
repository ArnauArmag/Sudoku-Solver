
# Sudoku Solver Readme

## Approach Explanation
### *Initial Modelling*
Following some reading about constraint satisfaction problems (CSP) I decided to use a common but effective algorithm to achieve a solution. The first step was to generalise the problem : 
* `X`, the set of variables
* `D`, the set of domains (one for each variable)
* `C`, the set of constraints that specify allowable combinations of values

I concluded that `X` would be each square in the sudoku board, The sudoku consists of a `9x9` grid so there are `81` variables in each problem. The naming convention used was *row*-*column*. The *rows* had lettering `{A,B,...,I}`, whilst the columns where numbered `{1,2,...,9}`. For example the first square (top-left) is mapped to the variable `A1`. Furthermore the set of domains is simple to start with, all variables will have the same domain in a blank grid which is the set of numbers `{1,2,...,9}`. During our implementation of the algorithm the domain of each variable will have to be reduced, this is due to the fact that some variables may already have a value assigned when we load the grid and others have their domain reduced by the different constraints. The last set-up step was the set of constraints. There are three main types of constraints in a sudoku:
1. Each row cannot have a repeated number. *( `9` constraints)*
2.  Each column cannot have a repeated number. *( `9` constraints)*
3.  Each of the `9`, `3x3` grids inside the sudoku cannot have a repeated number. *( `9` constraints)*

We can clearly see all the constraints have a similar format, consisting of `9` variables. There are a total of `27` constraints, all variables in the constraint must take a different value. A sudoku will have a valid solution if we are able to return a set of values which satisfies the following, for all the constraints:

* >![eq1](https://user-images.githubusercontent.com/60605841/109685927-abe36d80-7b79-11eb-85f0-0f4ca70473ba.jpg)


An example of each type of constraint we mentioned above are:
1. >![eq2](https://user-images.githubusercontent.com/60605841/109690031-d1727600-7b7d-11eb-9963-df1b50593ad5.png) -*row*
2. >![eq3](https://user-images.githubusercontent.com/60605841/109690086-ddf6ce80-7b7d-11eb-9c71-b11beb55bea7.jpg) -*column*
3. >![eq4](https://user-images.githubusercontent.com/60605841/109690139-ee0eae00-7b7d-11eb-9934-94be9495e398.jpg) -`3x3` *grid*

## *Algorithm Selection and Implementation*
Once I had constructed the basic data structures to have the sudoku loaded in and the initialise the set of variables along with their respective domain and construct the constraints, we had to be able to narrow down the set of possible values a variable can take. In other words we had to implement a constraint propagation algorithm. The constraint propagation algorithm to reduce the values a domain of each variable had was **Arc Consistency**. By definition the variables ![subscript](https://user-images.githubusercontent.com/60605841/109687308-0630fe00-7b7b-11eb-984b-66fbe87bc7af.jpg) and ![subscriptj](https://user-images.githubusercontent.com/60605841/109688940-bc491780-7b7c-11eb-9f96-42dca248a929.jpg) are *arc consistent* if for every value in the current domain ![dsubi](https://user-images.githubusercontent.com/60605841/109689724-8193af00-7b7d-11eb-9f2b-5371aedf768d.jpg) there is some value in the domain ![dsubj](https://user-images.githubusercontent.com/60605841/109689810-98d29c80-7b7d-11eb-8720-7f7e91b6fb75.jpg)
that satisfies the binary constraint in the arc (![subscript](https://user-images.githubusercontent.com/60605841/109687308-0630fe00-7b7b-11eb-984b-66fbe87bc7af.jpg),![subscriptj](https://user-images.githubusercontent.com/60605841/109688940-bc491780-7b7c-11eb-9f96-42dca248a929.jpg)). Once this arc consistency has been done there were three possible outcomes :
* The CSP contains a variable ![subscriptj](https://user-images.githubusercontent.com/60605841/109688940-bc491780-7b7c-11eb-9f96-42dca248a929.jpg)which has an empty domain (![dsubi](https://user-images.githubusercontent.com/60605841/109692276-3cbd4780-7b80-11eb-8edc-5463a71072a3.jpg)=ø), if this occurs our algorithm can immediately return  a sudoku grid full of `-1` as it will have no consistent solution.
* There is only one possible value for each variable which means our algorithm has reached a *goal state* and has finished. We will return the consistent solution achieved.
* We still have multiple possible values for variables. In this case we have to search for a solution, to do this we will use a **backtracking algorithm**.

### Backtracking Search
To keep things simple I started by testing a basic implementation of a backtracking algorithm. The strategy was to select the next unassigned variable, they were ordered by row and then column, (`A1` had the highest priority going down to `I9` with the lowest). Once the variable had been selected just tried all values in the domain of this variable and try to extend it to a solution in the CSP. If it found one, we returned it otherwise, just backtrack to the state where we started and try the next value, if we had found no solution and tried every value we would conclude that there was no consistent solution.

**Improvements**

As we were solving a CSP we could take advantage of their factored representation and therefore use domain independent solutions to improve the backtracking. With this taken into account I implemented a **Minimum-remaining-values** (MRV) heuristic, this made the algorithm choose the unassigned variable which had the least possible values. This means that it chooses the variable which is most likely to cause failure and hence reduce the amount of unnecessary searches. Arc consistency reduced the domains of variables before we began the search and MRV allowed us to improve efficiency when choosing which variable to start the search with. There was still a futher possibility of improvement by using inference during the search, this way we could infer domain reductions on new variables. When a variable ![subscript](https://user-images.githubusercontent.com/60605841/109687308-0630fe00-7b7b-11eb-984b-66fbe87bc7af.jpg) had been assigned **forward checking** allowed the algorithm to establish arc consistency with every unassigned variable ![subscriptj](https://user-images.githubusercontent.com/60605841/109688940-bc491780-7b7c-11eb-9f96-42dca248a929.jpg) which is connected to ![subscript](https://user-images.githubusercontent.com/60605841/109687308-0630fe00-7b7b-11eb-984b-66fbe87bc7af.jpg) by a constraint. This deleted any value from the domain ![dsubj](https://user-images.githubusercontent.com/60605841/109689810-98d29c80-7b7d-11eb-8720-7f7e91b6fb75.jpg) which was inconsistent with value chosen for ![subscript](https://user-images.githubusercontent.com/60605841/109687308-0630fe00-7b7b-11eb-984b-66fbe87bc7af.jpg). By using forward checking we found a more powerful way to compute the data neccesary for the MRV heuristic. 

## Other Ideas
The algorithm was able to solver all sudoku's in less than `0.5` seconds with the algorithm used which involved the arc consistency to propagate the constraints and then a backtracking search using forward checking and the MRV. In this section, I will give a brief overview of some algortihms I decided not to implement for different reasons, however they could achieve more efficiency or solve harder problems. Implementing a **PC-2** algorithm to do the constraint propagation instead of **AC-3** means we could solve some harder sudoku's from without the need of searching although the computational cost will increase. This is because *path consistency* looks at triples of variables instead of pairs as in *arc consistency*.A more powerful algorithm in the interleaving search and inference is the **MAC** *(Mantaining arc consistency)*. This will detect more inconsistencies than *forward checking* as it does a similar process but it does not recursively propagate the constraints when a domain has been revised. 

## References

Russell, S. and Norvig, P., Artificial Intelligence A Modern Approach. 