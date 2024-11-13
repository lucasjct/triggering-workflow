# triggering-workflow

Triggering workflow from another repository   


## Topics covered

1.  Create token with permissions to access repository and workflow.  

2. Save the token as a Secrets.

3. Create the workflow that will call the external workflow from another repository among the steps.  

4. Apply the correct workflow type to external repository called.   

5. Run action.  


### 1 Creating token: 

As we use the GitHub API, we need some token with necessary permissions to send events. For to do this, access __"Settings"__, on the top right side.   

![alt text](/images/image.png)    

After that, go to the left side and access respectively: __"Developer settings"__ > __"Personal access tokens"__  > __"Tokens (classic)"__  

Developer settings    

![alt text](/images/image-1.png)   


Tokens (classic)  

![alt text](/images/image-2.png)    

Select __"Generate new token (classic)"__ 


![alt text](/images/image-3.png)     

Select the permissions to access repository and workflow:  

![alt text](/images/image-5.png)  

Finally, click on "Generate Token"  and the token will be created. Copy this token, because we will use it on the next step:  ![alt text](/images/image-6.png)   


### 2 Save the token as a Secrets.  

The token saved, now we should attribute the token value to one secret.  
Go to the repository __Settings__.   


![alt text](image.png)  

Select __Secrets and Variables__ and click on __Actions__:  

![alt text](image-1.png)   


Now, click on button __"New reposistory secret"__.  And paste the token value, and give a name to this variable. Finally, click on __"Add secret"__ .  

![alt text](image-2.png)  

All done! Now, we can use this value into bearer token and send requests.   


### 3. Create the workflow that will call the external workflow from another repository   

Now, first lets create the step that will execute the request to another workflow:   

```bash
name: Send repository dispatch event

on:
  workflow_dispatch:

jobs:
  trigger-event:
    runs-on: ubuntu-latest
    steps:
      - name: Second fire event
        run: |
          curl -L \
            -X POST \
            -H "Accept: application/vnd.github+json" \
            -H "Authorization: Bearer ${{ secrets.TOKEN_ACCESS }}" \
            -H "X-GitHub-Api-Version: 2022-11-28" \
            https://api.github.com/repos/lucasjct/github-actions/dispatches \
            -d '{"event_type":"dispatch-event"}'  
```   

In the example above, we are using the GitHub API to trigger the workflow sending request to `https://api.github.com/repos/<OWNER-REPO>/<PROJECT-NAME>/dispatches`, to endpoint __/dispatches__. Note that we are using the TOKEN_ACCESS created on step before. We access it with the sintaxy `${{ secrets.TOKEN_ACCESS }}` .

 Now, we need apply to another repository the type `dispatch-event`.      


### 4 - Apply the correct workflow type to external repository called.    

Note below the type `[dispatch-event]` to `repository_dispatch`, this workflow will run when the request `https://api.github.com/repos/lucasjct/github-actions/dispatches`  be triggered.


```bash
name: First Workflow
on:
  repository_dispatch:
    types: [dispatch-event]
jobs:
  first-job:
    runs-on: ubuntu-latest # environments (runner)
    steps:
      - name: Print greeting and Good Bey
        run: | 
          echo "Hello World"
          echo "Good Bye"
```  

To make a workflow be triggered by another workflow, first we need follow the step 3 (config the curl with token and dispatch event), and apply this informations on workflow target: 


```bash
on:
  repository_dispatch:
    types: [dispatch-event]
```  

THis workflow will listen the called presents on `https://api.github.com/repos/lucasjct/github-actions/dispatches` .  


