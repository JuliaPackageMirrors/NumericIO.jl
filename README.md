# NumericIO.jl

[![Build Status](https://travis-ci.org/ma-laforge/NumericIO.jl.svg?branch=master)](https://travis-ci.org/ma-laforge/NumericIO.jl)

## Description

Improved support for formatting numeric data.

 - Includes facilities to display values using SI prefixes (`Y`, `Z`, `E`, `P`, `T`, `G`, `M`, `k`, `m`, &mu;, `n`, `p`, `f`, `a`, `z`, `y`)
 - Makes it easy to control the number of significant digits to display.

## Basic Usage

NumericIO.jl tries to provide the convenience of c++ `ios_base` configurability (ex: setting `ios_base::precision`) *without* modifying the output format of the base streaming object. Instead, NumericIO.jl uses the `FormattedIO` wrapper object to print data with the desired output format.

To obtain a string representation of a `Real` value using SI prefixes, one can use the `formatted` function:

	formatted(3.14159e-8, :SI, ndigits=3) # => "31.14n"

Similarly, one can generate a string using scientific notation with the following:

	formatted(3.14159e-8, :SCI, ndigits=3) # => "3.14×10⁻⁸"

Or using engineering notation (limiting to powers divisible by 3) with:

	formatted(3.14159e-8, :ENG, ndigits=3) # => "31.4×10⁻⁹"

To limit results to `ASCII` output, specify the `charset` keyword:

	formatted(3.14159e-8, :ENG, ndigits=3, charset=ASCIIString) # => "31.4E-9"

One might also prefer to create a convenience formatting function:

	SI(x) = formatted(x, :SI, ndigits=4)
	SI(3.14159e-9) # => "3.142n"
	SI(2.71828e12) # => "2.718T"

To print out multiple values, it is preferable (more efficient) to directly create a FormattedIO wrapper object:

	fio = formatted(STDOUT, :SI, ndigits=4) # => FormattedIO
	println(fio, 3.14159e-9) # => 3.142n
	println(fio, 2.71828e12) # => 2.718T
	...

## Advanced Usage

Lower-level structures of NumericIO can be used to fine-tune numeric output even further, if desired.  The following shows an example that approximates engineering notation using the `ASCII` characterset only:

	asciiexponentfmt = NumericIO.IOFormattingExpNum{ASCIIString}(
		"x10^", true, false, '+', '-', NumericIO.ASCII_SUPERSCRIPT_NUMERALS
	)
	fmt = NumericIO.IOFormattingReal(asciiexponentfmt,
		ndigits=4, decpos=0, decfloating=true, eng=true, minus='-', inf="Inf"
	)
	fio = formatted(STDOUT, fmt) # => FormattedIO
	println(fio, 3.14159e-8) # => 31.42x10^-9

It is also possible to generate the mantissa & exponent portions of a number separately.  This could be useful when displaying a plot's tick labels when using a common axis multiplier.  See implementation of `NumericIO.print_formatted(..., showexp=false)` and `NumericIO.print_formatted_exp(...)` for details.

## Known Limitations

 - Support for SI notation is limited between `y (1e-24)` and `Y (1e24)`.  Values beyond this range default to scientific notation.
 - SI notation displays `{1e9, 10e9, 100e9}` as `{1G, 10G, 100G}`.  It would be possible to reconfigure NumericIO to arbitrarily display `{1G, 10G, 0.1T}`, or `{1G, 0.01T, 0.1G}`, or ...
 - Algorithms are likely be a bit more complicated than absolutely necessary.  It would be nice to simplify/optimize the code as much as possible.
 - Does not support arbitrary bases (ex: `1x2^8`).

### Compatibility

Extensive compatibility testing of NumericIO.jl has not been performed.  The module has been tested using the following environment(s):

 - Linux / Julia-0.4.5
