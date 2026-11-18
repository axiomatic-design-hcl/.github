# Axiomatic Design HCL

This organization makes RequirementsHCL, a text language for software designs, and `adhcl`, a tool that checks designs written in it. A design file has the `.ad.hcl` suffix. The file states what the software must do, which design choices do it, and how the choices affect each other. The tool reads the files and tells you where the design is coupled, incomplete, or unproven.

The website is [axiomatic-design-hcl.github.io](https://axiomatic-design-hcl.github.io).

## The method in short

Nam P. Suh developed Axiomatic Design to turn needs into a design and to evaluate the result. Kim, Suh, and Kim applied it to software in 1991. The method uses these terms:

| Term | Meaning |
|---|---|
| Need | A problem that a person or group has. |
| Functional requirement (FR) | An outcome that the software must give. |
| Design parameter (DP) | A design choice that controls one FR. |
| Process variable (PV) | How the team builds a DP, for example an existing library. |
| Design matrix | A table with one row for each FR and one column for each DP. A cell says whether a change to the DP changes the FR. |

The Independence Axiom asks you to keep the FRs independent. In a good design, each DP changes its own FR and no FR that you set earlier. The Information Axiom asks you to select, from the independent designs, the design with the highest probability of success. That second step needs evidence, for example test results.

A matrix cell has one of three values. `X` means that the DP changes the FR. `0` means that it does not. `?` means that nobody knows yet. RequirementsHCL keeps `?` as unknown. It never treats an unknown effect as zero.

## The repositories

| Repository | Contents | Read it when |
|---|---|---|
| [spec](https://github.com/axiomatic-design-hcl/spec) | The language reference, the field schema, the assessment rules, the finding codes, examples, and conformance tests. | You want the exact rules of the language. |
| [adhcl](https://github.com/axiomatic-design-hcl/adhcl) | The command-line tool, written in Go. It checks a design and writes a text report or an HTML page. | You want to check your own design files. |
| [axiomatic-design-plugin](https://github.com/axiomatic-design-hcl/axiomatic-design-plugin) | A Claude Code plugin with two skills. One skill applies the method. The other skill writes the result as `.ad.hcl` files. | You want an AI assistant to help with the design. |
| [axiomatic-design-hcl.github.io](https://github.com/axiomatic-design-hcl/axiomatic-design-hcl.github.io) | The website and the course. | You want to learn the language step by step. |

## Get started

You need Go 1.25 or later and a terminal.

1. Clone the tool: `git clone https://github.com/axiomatic-design-hcl/adhcl.git`
2. Go into the folder: `cd adhcl`
3. Build the tool: `make build`. The binary goes to `bin/adhcl`.
4. Check the small example: `bin/adhcl validate examples/minimal.ad.hcl`
5. Make an HTML page from the example: `bin/adhcl diagram -o thermostat.html examples/minimal.ad.hcl`
6. Open `thermostat.html` in a browser.

The page has one tab for each reader. The business view shows needs and limits. The product view shows the FRs and how to accept them. The architecture view shows the DPs and the design matrices. The engineering view shows modules and the order in which they run. The other tabs show change impact, open risks, and a guide to the method.

## A small example

This file states one need, one FR, and one DP that controls the FR:

```hcl
design "thermostat" {
  boundary = "Keep one room near its set point."
}

need "steady_room" {
  statement = "Keep the room at a steady temperature."
}

fr "hold_setpoint" {
  need      = [need.steady_room]
  statement = "Keep the room within one degree Celsius of the set point."
}

dp "heater_control" {
  controls       = [fr.hold_setpoint]
  responsibility = "Switch the heater on below the set point and off above it."
  decision       = "selected"
}

influence {
  from      = [dp.heater_control]
  to        = fr.hold_setpoint
  effect    = "material"
  mechanism = "The switching rule sets how closely the room stays at the set point."
}
```

The tool reports four warnings for this file. The FR has no measurable acceptance criterion and no test scenario. No module builds the DP. No review supports the stated effect. Each report line names a finding code, a location, and the problem. The full example in the `adhcl` repository adds criteria and a second FR.

## Learn more

1. Take the course at [axiomatic-design-hcl.github.io/learn.html](https://axiomatic-design-hcl.github.io/learn.html). It builds one model from an empty file, and you run the tool after each lesson. It takes about three hours.
2. Read the [language reference](https://github.com/axiomatic-design-hcl/spec/blob/main/docs/language.md) for the syntax.
3. Read the [field schema](https://github.com/axiomatic-design-hcl/spec/blob/main/docs/schema.md) for each block and field.
4. Read the [assessment contract](https://github.com/axiomatic-design-hcl/spec/blob/main/docs/assessment-contract.md) to learn how the tool classifies a matrix.

## What a result means

A report with zero errors means that the tool completed its checks. It does not prove that the design is independent, built, or correct. The tool shows each claim with its source: the model states it, the author marks it as an inference, the tool assessed it, or the model has no record of it.

## License

All repositories use the Apache License 2.0.
