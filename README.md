## 🔥  Test Input Generation

```python
TAGS = ["Base", "Corner", "Time", "Space"]
problems = load_dataset("evalplus/humanevalplus")["test"]
# problems = load_dataset("evalplus/mbppplus")["test"]

# heler function
def extract_nparameter_mbpp(header: str) -> Tuple[str, List[str]]:
    entry_point = re.search(r"def\s+(\w+)", header).group(1)
    pattern = r"\((.*?)\)"
    match = re.search(pattern, header)
    if match:
        parameters_str = match.group(1)
        parameter_names = [
            param.split(":")[0].strip() for param in parameters_str.split(",")
        ]
        return entry_point, parameter_names
    else:
        return entry_point, []


def extract_nparameter_humaneval(header: str) -> List[str]:
    pattern = r"\((.*?)\)"
    match = re.search(pattern, header)
    if match:
        parameters_str = match.group(1)
        parameter_names = [
            param.split(":")[0].strip() for param in parameters_str.split(",")
        ]
        return parameter_names
    else:
        return []


def get_input(args, problems):
    data = json.load(open("./humaneval_sleuth.json", "r"))
    inputs = defaultdict(dict)
    for task_id, problem in tqdm(problems.items()):
        try:
            exec_globals = {}
            exec(testcase, exec_globals)
            if args.dataset == "humaneval":
                solution = problem["prompt"]
            elif args.dataset == "mbpp":
                solution = problem["code"]
            header = get_header(solution)
            if args.dataset == "mbpp":
                _, params = extract_nparameter_mbpp(header)
                nparams = len(params)
            else:
                nparams = len(extract_nparameter_humaneval(header))
            for tag, cases in zip(TAGS, exec_globals["testcases"]):
                inputs[task_id][tag] = (
                    [[case] for case in cases] if nparams == 1 else cases
                )
        except Exception as e:
            print(f"Error in getting input for {task_id}: {e}")
            print(f"{traceback.format_exc()}")
            exit(0)
    return inputs
```

## Evaluation

We use the [EvalPlus](https://github.com/evalplus/evalplus) framework for evaluation.