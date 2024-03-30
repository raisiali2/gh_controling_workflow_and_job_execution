# controlling workflow and job execution
## module introduction
## understanding conditional steps & jobs
in here there is
- jobs
  - conditional execution via if field
- steps
  - conditional execution via if field 
  - ignore errors via continue-on-error field
  on the the two above fields we can evaluate condition via expressions

## demo project setup
the project contains lint, test, build and deploy the code
like build jobs build the workflow and deploy jobs download the built image then deploy to the host machine.  
yet there is no if conditions.
## understanding a (potential) problem: a failing step
the default behavior of github is like if a test fail all the dependent test will fail.
for example:
if i put condition bellow the the test report such as
```github action
- name: generate test report
  if:
```
in the above example the if condition wrote bellow the test run, it means when the test fail the condition i wrote bellow will not execute  
because it is bellow the run test action.  
in github the the action will run which is parallel and not depend on the success of previous action, such as lint which not depend on the test will execute and run even if the test action failed.  
in conclusion putting if condition in above case wont be execute because it is after test action and test action already failed.
```json
describe('MainContent', () => {
  it('should render a button', () => {
    render(<MainContent />);

    expect(screen.getByRole('button')).toBeInTheDocument();
  });

  it('should show the help area after clicking the button', async () => {
    render(<MainContent />);

    const button = screen.getByRole('button');
    await userEvent.click(button);
    expect(screen.getByTestId('help-are')).toBeInTheDocument();
  });
});

```
remove `a` from `('help-are'))` and push changes

## controlling execution via `if`
now lets add `if` condition, specifying that if previous test action failed, then generate the test report.  
but still it is not enough, even by adding the `if` condition we still see that the test action failed and report does not generated,  
because still we did not change the default behavior of github action to do that we need to use `failure()` function.

## working with special conditional function
there are 4 conditional functions in github
1. `failure()`: returns true when any previous step or job failed
2. `success()` returns true when none of the previous steps have failed
3. `always()` causes the step to always execute, even when cancelled
4. `cancelled()` returns true if the workflow has been cancelled

## conditional jobs
bellow is an example of `job-level` condition, needs sets the dependency to the other jobs.
```yml
# job level condition
  report:
    needs: [lint, deploy]
    if: failure()
    runs-on: ubuntu-latest
    steps:
      - name: output information
        run: |
          echo "something went wrong"
          echo "${{ toJson(github) }}"  
        # output the github context object
```

![condition job-level](image.png)

## more `if` examples
in here we use `if` condition in the step-level to optimize caching such as instead of caching the global `npm` in the `.npm` this time we cache the `components` folder.
```yml
- name: Cache dependencies
        id: cache
        uses: actions/cache@v3
        with:
          # path: ~/.npm
          path: node_modules
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        if: steps.cache.outputs.cache-hit != 'true'
        run: npm ci
```

![alt text](image-1.png)

as in this time all the previous steps ran successfully and there is no failure the `report:` job did not evaluate or executed.
```yml
  # job level condition
  report:
    needs: [lint, deploy]
    if: failure()
    runs-on: ubuntu-latest
    steps:
      - name: output information
        run: |
          echo "something went wrong"
          echo "${{ toJson(github) }}"  
        # output the github context object
```

in this 

## ignoring errors & failures with `continue-on-error`

example continue with error:
```yml
     - name: Test code
        continue-on-error: true
        id: run-tests
        run: npm run test
```
![continue with error](image-2.png)

example of test broken:
```yml
      - name: Test code
        id: run-tests
        run: npm run test
      # test report make sense when we have failure report 
      # there for make if condition for that
      # like if the step run test failed     
      - name: Upload test report
        if: failure() && steps.run-tests.outcome == 'failure'
        # upload test report 
        uses: actions/upload-artifact@v3
        with:
          name: test-report
          path: test.json
```
![test broken](image-3.png)

`if check:` for executing some selective workflow


## understanding and using matrix strategies
is used one a specific job run with different configuration

```yml
name: matrix demo
on: push
jobs:
    build:
    # to run same job with different configuration use matrix
    # it might be one of the node-version does not support all the features needed
    # in this case we use continue with error because of one node other nodes evaluation or execution dont stop
        continue-on-error: true
        strategy:
            matrix:
                node-version: [12, 14, 16]
                operating-system: [ubuntu-latest, windows-latest]
        runs-on: ${{ matrix.operating-system }}
        steps:
          - name: get code
            uses: actions/checkout@v3
          - name: install nodejs
            uses: actions/setup-node@v3
            with: 
                node-version: 14
          - name: install dependencies
            run: npm ci
          - name: build project
            run: npm run build

```
![matrix workflow](image-4.png)

## including and excluding values (matrix strategy)
the include and exclude is another two features of matrix below is example
```yml
name: matrix demo
on: push
jobs:
    build:
    # to run same job with different configuration use matrix
    # it might be one of the node-version does not support all the features needed
    # in this case we use continue with error because of one node other nodes evaluation or execution dont stop
        continue-on-error: true
        strategy:
            matrix:
                node-version: [12, 14, 16]
                operating-system: [ubuntu-latest, windows-latest]
                        # add two others matrix features
                include:
                    - node-version: 18
                      operating-system: ubuntu-latest
                exclude:
                    - node-version: 12
                      operating-system: windows-latest

        runs-on: ${{ matrix.operating-system }}

        steps:
          - name: get code
            uses: actions/checkout@v3
          - name: install nodejs
            uses: actions/setup-node@v3
            with: 
                node-version: 14
          - name: install dependencies
            run: npm ci
          - name: build project
            run: npm run build


```
![include and exclude](image-5.png)

## saving time and code with reusable workflows

## adding inputs to reusable workflows & secrets

## reusable workflows & secrets

## reusable workflows outputs

## module summary
