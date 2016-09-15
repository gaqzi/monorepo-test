# Pipeline behaviors for a monorepo

This is a repo setup for testing how different CI/CD servers work with
monorepos. The idea is to see whether they can smartly deal with
dependencies between subprojects/modules and only run their pipelines
as the dependencies pass.

The project that is being setup will have dependencies like this:

There are two sites:

src/sites/{australia, sweden}

These sites depend on mostly the same extensions, but they can be
different for different countries.

Dependency per site in order
australia:
  1. pricing
  2. theme--base

sweden:
  1. pricing
  2. theme--base
  3. theme--sweden

The sites have no relationship with each other, apart from depending on
the same extensions.

## What should happen?

Take australia from the above example. If a commit goes into pricing then:

- The tests for pricing should be run, if green
- The tests for theme--base should be run, if green
- The tests for australia should be run, if green
- The deploy to staging for should be run

Meanwhile since pricing was a dependency for sweden as well:

- The tests for pricing should be run, if green
- The tests for theme--base should be run, if green
- The tests for theme--sweden should be run, if green
- The tests for sweden should be run, if green
- The deploy to staging for sweden should be run

If there's a failure in any step for either Australia or Sweden then
no subsequent step would be run.

## How the tests work

Because this is just intended to mimic the behavir of the real world the
test script will just exit 0 or 1 depending on what we want for the current
scenario.

# Test cases

1. Initial run
   All code is green and being run for the first time.
   Expected outcome: All subprojects gets run in order until green
2. A non-breaking change to theme--sweden
   Expected outcome: theme--sweden runs, goes green and then triggers
   sweden that goes green.
3. A non-breaking change to pricing
   Expected outcome: pricing goes green and all subsequent downstreams

## GoCD - 16.9.0

1. Passed and succeeded as expected
2. GoCD picks up that there has been a material change. No run is scheduled
3. GoCD picks up that there has been a material change. No run is scheduled

I'm assuming the problem lies with how Go's fan-in works. Looking at
the developer documentation for [Corner Cases (section 5.2.4)][corner-cases]
I think this is exactly the problem I'm hitting.

As I have blacklists (albeit inverted so they're whitelists) that
targets different parts of the source tree.

[corner-cases]: https://developer.go.cd/16.9.0/5/5.2.html#5-2-4-a-name-corner-cases-a-corner-cases
