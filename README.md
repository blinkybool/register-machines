# Register Machines

`rm.luau` simulates [register machines](https://webspace.science.uu.nl/~ooste110/syllabi/compthmoeder.pdf) with lua tables.

I think the composition, primitive-recursion, minimisation operations are correct.
But you don't prove things like this in lua. Use lean for that.
These register machines run wild and free with little type constraints.
Batteries and disk space not included.

```lua
-- Programs are lists of commands

-- This one empties register 1 into register 2, then halts
local program = parseProgram {
	"R1-=>2,3",
	"R2+=>1",
}

-- A register machine containers program
-- and an initially empty (numeric) table of registers
local rm = RM.new(program, {2,5,7})

-- You can run a register machine (it might not halt!)
-- and optionally print the state at each step
rm.run(true)
--[[ Output
	1: R1-=>2,3              2,5,7
	2: R2+=>1                1,5,7
	1: R1-=>2,3              1,6,7
	2: R2+=>1                0,6,7
	1: R1-=>2,3              0,7,7
--]]

-- A register machine computes a partial function f:ℕ^k ↪ ℕ
-- iff ∀x ∈ dom(f) ⊆ ℕ^k
-- `rm.registers = {x1,...,xk}; rm.run()`
-- halts and leaves f(x) in the first register

-- RM.compute does this on a given `input: {number}`, and returns the value
print(RM.new(program).compute({1,2,3})) --> 0

-- PR proves the class of register machine programs is closed under
-- composition, primitive recursion and minimisation.

-- z(x) = 0
local z = PR.zero

-- s(x) = x+1
local s = PR.succ

-- p3(a,b,c,d,e) = c
local p3 = PR.proj(3)

-- f(x,y) = h(g1(x,y), g2(x,y), g3(x,y))
local f = PR.compose(h, g1, g2, g3)

-- f(0, xs) = g(xs)
-- f(y+1, xs) = h(y, f(y, xs), xs)
local f = PR.primrec(h, g)

-- f(xs) = min{y | h(y,xs) == 0}
local f = PR.primrec(h)

-- Arithmetic functions are easily composed from these operations
local o = PR.compose
local rec = PR.primrec

-- x + y
local add = rec(p1, o(s, p2))
-- x * y
local mul = rec(z, o(add, p2, p3))
-- y ^ x
local flippedPow = rec(o(s,z), o(mul, p2, p3))
-- x ^ y
local pow = o(flippedPow, p2, p1)
```
