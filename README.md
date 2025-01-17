# LogFixPoint16s.jl
[![CI](https://github.com/milankl/LogFixPoint16s.jl/actions/workflows/CI.yml/badge.svg)](https://github.com/milankl/LogFixPoint16s.jl/actions/workflows/CI.yml)
[![DOI](https://zenodo.org/badge/254140322.svg)](https://zenodo.org/badge/latestdoi/254140322)

Provides LogFixPoint16 - a (software-implemented) 16-bit [logarithmic fixed-point number](https://en.wikipedia.org/wiki/Logarithmic_number_system) format with adjustable numbers of integer and fraction bits.

### Example use

```julia
julia> using LogFixPoint16s
julia> v = LogFixPoint16.(rand(Float32,5))
5-element Array{LogFixPoint16,1}:
 LogFixPoint16(0.8925083)
 LogFixPoint16(0.4919428)
 LogFixPoint16(0.69759846)
 LogFixPoint16(0.25616693)
 LogFixPoint16(0.57248604)

julia> sum(v)
LogFixPoint16(2.9139352)
```

### Features

Exports `LogFixPoint16, iszero, isnan, signbit, zero, nan, floatmin, floatmax, one, -, inv, *, / , +, -, sqrt, nextfloat, prevfloat, ==, <=, >, >=, show, bitstring` as well as conversions to and from `Float64, Float32, Float16, Int`.

Although `LogFixPoint16` is always a 16-bit format, the number of fraction bits (in exchange for integer bits) can be adjusted between 7 and 11. For 7 fraction bits, `LogFixPoint16` has a similar dynamic range-precision trade-off as `BFloat16`; 10 fraction bits are similar to `Float16`.

```julia
julia> LogFixPoint16s.set_nfrac(7)
┌ Warning: LogFixPoint16 was changed to 8 integer and 7 fraction bits.
└ @ Main.LogFixPoint16s ~/git/LogFixPoint16s.jl/src/LogFixPoint16s.jl:24
```
Furthermode the rounding mode can be changed from round-to-nearest in linear space (default) to log space
```julia
julia> LogFixPoint16s.set_rounding_mode(:log)
┌ Warning: LogFixPoint16 rounding mode changed to round to nearest in log-space.
└ @ LogFixPoint16s ~/.julia/packages/LogFixPoint16s/TGYbV/src/change_format.jl:48
```
The two arguments `:lin` and `:log` are allowed.

### Theory

A real number `x` is encoded in LogFixPoint16 as

```
x = (-1)^s * 2^k
```
with `s` being the sign bit and `k = i+f` the fixed-point number in the exponent, consisting of a signed integer `i` and a fraction `f`, which is defined as the significant bits for floating-point numbers. E.g. the number `3` is encoded as

```julia
julia> bitstring(LogFixPoint16(3),:split)
"0 1000001 10010110"
```
The sign bit is `0`, the sign bit of the signed integer is `1` (meaning + due to the biases [excess](https://en.wikipedia.org/wiki/Signed_number_representations#Comparison_table) representation) such that the integer bits equal to `1`. The fraction bits are 1/2 + 1/16 + 1/64 + 1/128. Together this is

```
0 1000001 10010110 = +2^(1 + 1/2 + 1/16 + 1/64 + 1/128) = 2^1.5859375 ≈ 3
```
The only exceptions are the bitpatterns `0x0000` (zero) and `0x8000` (Not-a-Real, NaR). The smallest/largest representable numbers are (6 integer bits, 9 fraction bits)

```julia
julia> floatmin(LogFixPoint16)
LogFixPoint16(2.3314606e-10)

julia> floatmax(LogFixPoint16)
LogFixPoint16(4.2891566e9)
```
 
### Decimal precision

Logarithmic fixed-point numbers are placed equi-distantly on a log-scale. Consequently, their decimal precision is perfectly flat throughout the dynamic range of representable numbers. In contrast, floating-point numbers are only equi-distant in logarithmic space when the significand is held fixed; the significant bits, however, are linearly spaced.

As a consequence there is no rounding error for logarithmic fixed-point numbers in multiplication, division, power of 2 or square root - similarly as there is no rounding error for fixed-point numbers for addition and subtraction - as long as no over or underflow occurs.

![decimal precision](figs/decimal_precision.png?raw=true "decimal precision")

LogFixPoint16 with 10 fraction bits have a similar decimal precision / dynamic range trade-off as Float16, and 7 fraction bits are similar to BFloat16. However, these decimal precision only apply to additions, as multiplications are rounding error-free. `LogFixPoint16s.jl` also allows additionally for 8,9 or 11 fraction bits, which are not shown.

### Benchmarks

Although `LogFixPoint16s` are software-emulated, they are considerably fast. Define some matrices

```julia
julia> using LogFixPoint16s, BenchmarkTools
julia> A = rand(Float32,1000,1000);
julia> B = rand(Float32,1000,1000);
julia> C,D = Float16.(A),Float16.(B);
julia> E,F = LogFixPoint16.(A),LogFixPoint16.(B);
```
And then benchmark via `@btime +($A,$B):` and so on. Then relative to `Float64` performance for addition:

| Operation           | Float64 | Float32 | BFloat16 | Float16 | LogFixPoint16 |
| ------------------- | ------- | ------- | -------- | ------- | ------------- |
| Addition (+)        | 1200μs  |   500μs |   400μs  | 1800μs  | 6500μs        |
| Multiplication (.*) | 1200μs  |   500μs |   400μs  | 2500μs  | 250μs         |
| Power (.^2)         |  700μs  |   300μs |  2800μs  | 1000μs  | 700μs (*)     | 
| Square-root (sqrt.) | 1800μs  |   900μs |  1800μs  | 1200μs  | 170μs         |

On an Intel i5 (Ice Lake). (*) via `power2`.

### Installation

`LogFixPoint16s.jl` is registered in the Julia Registry, so simply do

```julia
julia> ] add LogFixPoint16s
```
where `]` opens the package manager.

### Citation

If you use this package in your work please credit us by citing

> Klöwer M, PV Coveney, EA Paxton, and TN Palmer. _Periodic orbits in chaotic systems simulated at low precision_. **Scientific Reports** 13, 11410 (2023). DOI:[10.1038/s41598-023-37004-4](https://doi.org/10.1038/s41598-023-37004-4)
