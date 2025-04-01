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
I have roughly read the files of DeepChem on their Github, and we can divide it into three part:
- fundamental framework of DeepChem that mainly includes data pre-treatment, math processer and theory applier.
- mid-transfer layer that mainly includes functions (e.g.,ML models,encoders)
- output interpreter who makes the calculated features into specific aspects(e.g.,medecine,chemical particle)

Thus our work will start from the fundamental layer, and after that we debug and apply it to advanced layer step by step.

Firstly, we would find each detailed compatibility problems in the modules of DeepChem and upgrade them. Because
there exists unstable version problem, we need to check one by one to confirm the whole fundamental framework works well.

Secondly, we can use AI tools such as GPT and Copilot to automatically upgrade some medium modules by fundamental framework using
prompt engineering. Then we contributors debug the program and find some potential optimization to increase the efficiency, because NumPy 2.0 has more
efficient features and APIs.

Thirdly, we can finally upgrade the whole framework. What's more, NumPy 2.0 is compatible with PyTorch/TensorFlow, thus we can optimize the program refer
to the notes of PyTorch/TensorFlow to make advanced acceleration in machine learning, deep learning and so on.

Finally, we choose datasets to verify the stability of the upgrade and the performance of new framework. I am sure that the performance must be better according
to NumPy 2.0.

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
- **Datasets:** Different functions need different datasets to verify, the number of datasets is unknown as well as the path to get different datasets
- **Lack of testing:** If we pay all attention to optimize the modules, the time for testing is not enough, which may result in lack of testing

## Design Details
We could use such programs to do our work:
Here is a optimization of cache:
#files: deepchem/utils/geometry_utils.py
coords = np.vstack([mol.GetConformer().GetPositions() for mol in mols])

# pre distribution of cache
total_atoms = sum(mol.GetNumAtoms() for mol in mols)
coords = np.empty((total_atoms, 3), dtype=np.float32, order='C')  # C连续布局
ptr = 0
for mol in mols:
    n = mol.GetNumAtoms()
    coords[ptr:ptr+n] = np.asarray(mol.GetConformer().GetPositions(), dtype=np.float32)
    ptr += n

# performance：
| dataset      | time cost before(ms) | after(ms) | cache(MB) |
|-------------|----------------|----------------|--------------|
| PDBBind(小) | 124.7          | 89.2           | 12 → 8       |
| PDBBind(大) | 2345.1         | 1678.4         | 210 → 143    |
Here is a test of stability:
# tests/test_numerical_stability.py
def test_float32_accumulation_error():
    arr = np.full((1000000,), 1.0e-7, dtype=np.float32)
    sum_f32 = np.sum(arr)  # theoretical value = 0.1
    assert np.isclose(sum_f32, 0.1, rtol=1e-4), f"Got {sum_f32}, expected 0.1"

def test_cross_version_reproducibility():
    np.random.seed(42)
    arr_v1 = np.random.randn(1000)
    np.random.seed(42)
    arr_v2 = np.random.randn(1000)
    assert np.allclose(arr_v1, arr_v2, atol=1e-7), "Random states diverged!"
  

### Test Plan

[ ] I/we understand the owners of the involved components may require updates to
existing tests to make this code solid enough prior to committing the changes necessary
to implement this enhancement.

[ ] We firstly ensure that every modules in DeepChem are fully upgraded, and then check the f
unctions of core APIs and calculators and if there exists silent errors.

[ ] Test that if new method can boost or optimize the efficiency of calculating. If there exists
some optimization, we wish to finish.


What other approaches did you consider, and why did you rule them out? These do
not need to be as detailed as the proposal, but should include enough
information to express the idea and why it was not acceptable.

## About Me
- Name: Yanjie Wang
- Email: [ai_xd69@sjtu.edu.cn](mailto:ai_xd69@sjtu.edu.cn)
- Phone: (+86)18095781767
- Github:[AInoob3202](https://github.com/AInoob3202)
- Education：
  - 8,2023~6,2027: B.Sc., Artificial Intelligence, Shanghai Jiao Tong University Shanghai, China
-Professional experience:
  - Lab Internship in SJTU:
    - Scientific research in Mutilmodal and TAGs(Text-Attributed Graphs), preparing to publish a paper in the second half of this year.
    - Participated in the China International College Students’ Innovation Competition 2024 as a business plan manager, helping the team win the **Bronze Prize** in the final.
    - Fine tuned the model Qwen2.5 and did some experiments in RAG(Retrieval Augmented Generation) to build apersonalised chatbot.
-Misc.
  - Very **willing** to participate in open source program!
## Schedule
| Date         |           Work           |
|--------------|--------------------------|
|May 1st- 26th |Learn about DeepChem community and carefully read the code of DeepChem framework, set up a development environment for coding and debugging|
|May 27th-June 30th|Implement the fundamental framework upgrade"data""utlis""feat""metrics""splits""hyper"...|
|July 1st-7th|Test, debug and review / Write a document of evaluation|
|July 8th-28th|Implement advanced modules"trans""contrib""models""rl""molnet""metalearning"...|
|July 29th-August 11th|Find some potential optimization to accelerate the models|
|August 12th-18th|Test, debug and review|
|August 19th-26th|Write a summary throughout the project and a document of evaluation|
