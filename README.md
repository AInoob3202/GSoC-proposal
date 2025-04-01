# Better NumPy 2.0 Upgrade to DeepChem

## Summary
The migration of DeepChem from NumPy <2.0 to 2.0 may results in many compatibility problem.
We are concerned to resolve API incompatibilities, refactor deprecated patterns, optimize 
numerical computation performance, fix little syntax bugs and support third-party 
dependency coordination. 

## Motivation
- **Stability:** Latest version of NumPy is not stable enough, which needs us to fix
- **Better Performance:** Abandon inefficient approaches and use new features of NumPy 2.0 to do better 
- **Ecosystem:** PyTorch/TensorFlow 2.x now require NumPy 2.0 compatibility
- **Security:** Critical vulnerabilities in older NumPy versions (CVE-2023-52425)
- **Support on GPU:** New features accelerate GPU efficiency that we must focus on

### Goals
- Core API migration(considering both removals and new features)
- Performance optimization, e.g., using np.block(≥1.8x speedup verified)
- Global transfer(make all essential calculation modules 0% loss)
- Zero silent errors in type conversions

### Non-Goals
- Backport NumPy 2.0 features to older versions
- Optimize non-core computational pathways
- Support EOL NumPy versions (<1.21)

## Proposal

<!--
This is where we get down to the specifics of what the proposal actually is.
This should have enough detail that reviewers can understand exactly what
you're proposing, but should not include things like API designs or
implementation. What is the desired outcome and how do we measure success?.
The "Design Details" section below is for the real
nitty-gritty.
-->

### Risks and Mitigations
#### When developing
- **Implicit *dtype=object* failures:** Add context-aware *try-except* blocks
- **C extension segfaults:** Valgrind + GDB stack tracing
- **Third-party version locks:** Build dynamic version sniffers (e.g., tensorflow-numpy-adapter)
<!--
What are the risks of this proposal, and how do we mitigate? Think broadly.
For example, consider both security and how this will impact the larger
Kubeflow ecosystem.
How will security be reviewed, and by whom?
How will UX be reviewed, and by whom?
Consider including folks who also work outside the SIG or subproject.
-->
#### When testing
- **Datasets** 

## Design Details

<!--
This section should contain enough information that the specifics of your
change are understandable. This may include API specs (though not always
required) or even code snippets. If there's any ambiguity about HOW your
proposal will be implemented, this is the place to discuss them.
-->

### Test Plan

[ ] I/we understand the owners of the involved components may require updates to
existing tests to make this code solid enough prior to committing the changes necessary
to implement this enhancement.

[ ] We firstly ensure that every modules in DeepChem are fully upgraded, and then check the f
unctions of core APIs and calculators and if there exists silent errors.

[ ] Test that if new method can boost or optimize the efficiency of calculating. If there exists
some optimization, we wish to finish.

#### Prerequisite testing updates

<!--
Based on reviewers feedback describe what additional tests need to be added prior
implementing this enhancement to ensure the enhancements have also solid foundations.
-->

#### Unit Tests

<!--
In principle every added code should have complete unit test coverage, so providing
the exact set of tests will not bring additional value.
However, if complete unit test coverage is not possible, explain the reason of it
together with explanation why this is acceptable.
-->

<!--
Additionally, try to enumerate the core package you will be touching
to implement this enhancement and provide the current unit coverage for those
in the form of:
- <package>: <date> - <current test coverage>
This can inform certain test coverage improvements that we want to do before
extending the production code to implement this enhancement.
-->

- `<package>`: `<date>` - `<test coverage>`

#### E2E tests
1
<!--
Describe what E2E tests will be added to ensure proper quality of the enhancement.
After the implementation PR is merged, add the names of the tests here.
-->

#### Integration tests

<!--
Describe what tests will be added to ensure proper quality of the enhancement.
After the implementation PR is merged, add the names of the tests here.
-->

### Graduation Criteria

<!--
This section is optional until Kubeflow has formally defined graduation criteria,
feature gates, and a deprecation policy.

Clearly define what it means for the feature to be implemented and
considered stable.
If the feature you are introducing has high complexity, consider adding graduation
milestones with these graduation criteria:
- [Maturity levels (`alpha`, `beta`, `stable`)][maturity-levels]
- [Feature gate][feature gate] lifecycle
- [Deprecation policy][deprecation-policy]
[feature gate]: https://git.k8s.io/community/contributors/devel/sig-architecture/feature-gates.md
[maturity-levels]: https://git.k8s.io/community/contributors/devel/sig-architecture/api_changes.md#alpha-beta-and-stable-versions
[deprecation-policy]: https://kubernetes.io/docs/reference/using-api/deprecation-policy/
-->
What other approaches did you consider, and why did you rule them out? These do
not need to be as detailed as the proposal, but should include enough
information to express the idea and why it was not acceptable.
-->
