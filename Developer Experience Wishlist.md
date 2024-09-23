So far, we have focused primarily on getting our container images ready for deploying to production, and haven't focused on the developer experience.

## Key DevX Features:

1. **Easy/simple to set up:** Using docker compose, we can define the entire environment with a single yaml file. To get started, team members can issue a single command `make compose-up-build` or `make compose-up-build-debug` depending if they want to run the debugger or not.
    
2. **Ability to iterate without rebuilding the container image:** In order to avoid having to rebuild the container image with every single change, we can use a bind mount to mount the code from our host into the container filesystem. For example:
    

```yml
- type: bind
  source: ../05-example-web-application/api-node/
  target: /usr/src/app/
```

3. **Automatic reloading of the application:**
    
    - _React Client:_ We are using Vite for the react client which handles this handles this automatically
    - _Node API:_ We added nodemon as a development dependency and specify the Docker CMD to use it
    - _Golang API:_ We added a utility called `air` (https://github.com/cosmtrek/air) within `Dockerfile.dev` which watches for changes and rebuild the app automatically.
4. **Use a debugger:**
    
    - _React Client:_ For a react app, you can use the browser developer tools + extensions to debug. I did include `react-query-devtools` to help debug react query specific things. It is also viewed from within the browser.
    - _Node API:_ To enable debugging for a NodeJS application we can run the app with the `--inspect` flag. The debug session can then be accessed via a websocket on port `9229`. The additional considerations in this case are to specify that the debugger listen for requests from 0.0.0.0 (any) and to publish port `9229` from the container to localhost.
    - _Golang API:_ To enable remote debugging for a golang application I installed a tool called delve (https://github.com/go-delve/delve) within `./api-golang/Dockerfile.dev`. We then override the command used to run the container to use this tool (see: `docker-compose-debug.yml`)
5. **Executing tests:** We also need the ability to execute our test suites within containers. Again, we can create a custom `docker-compose-test.yml` overlay which modifies the container commands to execute our tests. To build the api images and execute their tests, you can execute `make run-tests` which will use the `test` compose file along with the `dev` compose file to do so.
    
6. **Continuous integration pipeline for production images**
    
7. **Ephemeral environment for each pull request**
    

In the following lessons we will address all of these!