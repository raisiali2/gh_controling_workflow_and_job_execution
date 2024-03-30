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

## understanding and using matrix strategies

## including and excluding values (matrix strategy)

## saving time and code with reusable workflows

## adding inputs to reusable workflows & secrets

## reusable workflows & secrets

## reusable workflows outputs

## module summary
