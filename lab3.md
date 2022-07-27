# Introduction to OpenShift

In this lab we are going to see how a developer and an admin can access the OpenShift Console in order to get an application deployed.

## Running Pacman from the Operator

1. Access the [OpenShift Console URL](https://console-openshift-console.apps.gva-workshop.sandbox858.opentlc.com/)
2. Login with `RHA` using the credentials shared during the session
3. Go to Developer perspective. If you get a prompt for a console tour press `Skip Tour`
4. Click on `Create a new project` and name it after yourself
5. On the left menu click on `+Add`
6. Under `Developer Catalog` click on `Operator Backed`
7. Click on `Pacman Game`
8. Click on `Create`
9. Give it a name and specify the number of replicas (defaults are great)
10. Click on Create
11. You will get presented with the application view, click on the item that starts with `pacman...`
12. Now change your view to Admin
13. Click on `Networking` -> `Routes`
14. Click on `Create Route`
15. Fill the form with a name for the route, select the pacman service and the port
16. Click on the URL

Congratulations! You have deployed an application from an Operator.

## Create application from Source

1. Loged in the OpenShift Console as a developer click on `+Add`
2. Under `Developer Catalog` click on `All services`
3. Searc for `Go` and click on the `Go` builder image
4. Click on `Create Application`
5. Select `1.16.7-ubi8` as `Builder Image version`
6. Click on `Try sample` under `Git Repo URL`
7. Under `General` name the application.
8. Under `Resources` make sure `Deployment` is selected
9. Under `Advanced options` make sure `Create a route to the Application` is selected
10. Click on `Create`
11. Back in the `Topology` view, click on the `Go` application
12. Under `Build` you will see the application being built, feel free to check the logs
13. Once the build is done, you can access your application using the `Route` under `Routes`, or the arrow sign under `Topology`

Congratulations! You have deployed an application from the source.

## Clean up

1. Go to `Administrator` view
2. Click on `Projects` under home
3. Click on the three dots to the far right of your project
4. Select `Delete Project`

## Admin Console

The instructor will show:

- `Overview`
- `Operators`
- `Observe`
- `Compute`
- `User Management`
- `Administration - Cluster Settings`