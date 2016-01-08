# MultiJuMP

[![Build Status](https://travis-ci.org/anriseth/MultiJuMP.jl.svg?branch=master)](https://travis-ci.org/anriseth/MultiJuMP.jl)

## Usage
As a usage example, we implement the test from
_Das and Dennis, 1998: Normal-boundary intersection: A new method for
generating the Pareto surface in nonlinear multicriteria optimization problems_:

```julia
using MultiJuMP, JuMP
using AmplNLWriter

m = MultiModel(solver = IpoptNLSolver())
@defVar(m, x[i=1:5])
@defNLExpr(m, f1, sum{x[i]^2, i=1:5})
@defNLExpr(m, f2, 3x[1]+2x[2]-x[3]/3+0.01*(x[4]-x[5])^3)
@addNLConstraint(m, x[1]+2x[2]-x[3]-0.5x[4]+x[5]==2)
@addNLConstraint(m, 4x[1]-2x[2]+0.8x[3]+0.6x[4]+0.5x[5]^2 == 0)
@addNLConstraint(m, sum{x[i]^2, i=1:5} <= 10)

obj1 = SingleObjective(f1)
obj2 = SingleObjective(f2)

multim = getMultiData(m)
multim.objectives = [obj1, obj2]
multim.ninitial = 5
solve(m, method = :NBI)
```

Plot with

```julia
f1arr = convert(Array{Float64}, [multim.paretofront[i][1] for i in 1:multim.ninitial])
f2arr = convert(Array{Float64}, [multim.paretofront[i][2] for i in 1:multim.ninitial])

nbi = plot(x=f1arr, y=f2arr, Geom.point,
               Guide.xlabel("f1"), Guide.ylabel("f2"))
```
![Pareto front example](./img/pareto_example.svg)


##TODO:
- Add bounds on the MultiObjective type
    * So we can ask to only search over subset of pareto front
    * __This seems to be causing problems__
- Test if multiobjective code works for three objectives
- Implement Struss test from Das&Dennis
- Add option to use inequality in NBI constraint?
- Tell travis to install Ipopt
- Create more tests (WS, 3 objectives)
