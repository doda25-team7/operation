### Week Q2.1 (Nov 10 - )

Group formation. 

### Week Q2.2 (Nov 17 - )

Group meeting on 18 nov
- Maciej: https://github.com/doda25-team7/model-service/pull/4, https://github.com/doda25-team7/model-service/pull/1, I worked on contenerization of our the app frontend and backend (F3-F6)
- Tess: https://github.com/doda25-team7/model-service/pull/3
I have worked on A1 and have contributed to the automation workflows, and versioning strategy. 
- Francisco: https://github.com/doda25-team7/lib-version/pull/3
I worked on A1, here contributed to feature F11 eleven of automatic version releasing in lib-version.
- Renyi: https://github.com/doda25-team7/lib-version/pull/1 , feature 1 and feature 2.

- Tobias https://github.com/doda25-team7/app-service/pull/3
I have worked on A1 and have contributed a versioning strategy for the app-service repository.

### Week Q2.3 (Nov 24 - )
- Francisco: https://github.com/doda25-team7/infrastructure/pull/1
Worked on all steps of 1.1 setting up provisioning and the infrastructure repo
- Maciej: https://github.com/doda25-team7/operation/pull/1, https://github.com/doda25-team7/operation/pull/2, https://github.com/doda25-team7/infrastructure/pull/2 ; I worked on improving the previous assignment(updating the README and doin docker compose (F7)). Additionally i implemented steps 13, 14 and 15 of A2
Group meeting on 25 nov
- Renyi: https://github.com/doda25-team7/infrastructure/pull/2 I did step 15 and 16 of assignment2. other parts are done by colleagues.

- Tobias: Was not able to make any significant contributions this week due to issues with the understanding and implementing theory from the lectures.
- Tess: Implemented the model training in CI https://github.com/doda25-team7/model-service/pull/5
### Week Q2.4 (Dec 1 - )
- Francisco: Will work on moving the docker containerization to kubernetes. F1 of the assignment 3
Due to sickness and being blocked by the completion of the previous assignment couldn't finish it. doda25-team7/infrastructure#8 

- Maciej: Fixed last assginment(problems with the dashboard): https://github.com/doda25-team7/infrastructure/pull/7

- Renyi: I did 1.3 and 1.4 of assignment 2. https://github.com/doda25-team7/infrastructure/pull/3

- Tobias: https://github.com/doda25-team7/infrastructure/pull/4
Also attempted to fix the problems with 1.3 and 1.4. 
However, ran into some problems with compatabiility with Linux due too restricted host-only network for virtualbox and an issue with the run join command which also seemed to happen only on my end (might also be linux related but no confirmed)
Will be adding the fixes to these problems to the readme beginning next week.
Also made a beginning making sure the code adheres to the A2 tips in 1.6: local provisioning and avoiding shell command.

- Tess: Spend a couple days trying to setup and debug the kubernetes instance. Made some changes in the cofiguration of Helm https://github.com/doda25-team7/infrastructure/pull/5

### Week Q2.5 (Dec 8 - )
- Francisco - working on migrating from docker-compose to kubernetes. Changed the Vagrat Ips to mach the virtual box allowed routing. 
https://github.com/doda25-team7/infrastructure/pull/8 -> Part 1 of assignemnt 3

- Tess - looked into the security implications of the Github Action version manager, and as a response to the recommendation, properly pinned the external personal repo we used in model-service https://github.com/doda25-team7/model-service/pull/6

- Tobias: https://github.com/doda25-team7/operation/pull/3
Worked on enabling monitoring using prometheus. Also finished up local provisioning of the ansible playbooks.

- Maciej: No new taks done this week as previous parts of assignment 3 were not finished. Made up for the contribution during christmas/winter break

### Week Q2.6 (Dec 15 - )
- Tess - Worked on the Ansible and Vagrant configuration and the Helm chart https://github.com/doda25-team7/operation/pull/4

- Renyi Working on alerting of assignment 3. https://github.com/doda25-team7/operation/pull/5

- Francisco - Working on Istio setup - this week converted helm chart setup to work with multiple operating systems https://github.com/doda25-team7/operation/pull/7

- Tobias: https://github.com/doda25-team7/operation/pull/3
Further progression with the monitoring setup using prometheus. Also worked on improving the helm chart and adding a MonitorController.java in app-service.

- Maciej: Started working on the Grafana dashboards for A3: https://github.com/doda25-team7/app-service/pull/8, fixed a problem with the sms model crashing: https://github.com/doda25-team7/model-service/pull/8. Additionally gave reviews for teammates.

### Week Q2.7 (Jan 5 - )
- Francisco - Working on Traffic Management - https://github.com/doda25-team7/operation/pull/7

- Renyi - Shadow launching - https://github.com/doda25-team7/operation/pull/9

- Tobias: https://github.com/doda25-team7/app-service/pull/11
Added 2 extra metrics to MetricController.java

- Maciej: Finished implementation of grafana dashboards: https://github.com/doda25-team7/operation/pull/6/files 

- Tess: Cleanup of the operation repository, and updating of various readmes: https://github.com/doda25-team7/operation/pull/8

### Week Q2.8 (Jan 12 - )
- Francisco - Done some deeper istio documentation and working on merging and updating shadow releases of the Model- was done by Renyi but it conflicts a lot with current main setup. Will try to rework it, add documentation as well. https://github.com/doda25-team7/operation/pull/11, also finished https://github.com/doda25-team7/operation/pull/7

- Tobias: https://github.com/doda25-team7/operation/commit/3c85d86864f6ee62dc08a234620d683c9cbb45a5
Added beginning draft of the deployment documentation.

- Maciej: Made necessary changes to the grafana dashboards to work with the new istio setup: https://github.com/doda25-team7/operation/pull/10. Fixed a small issue with Prometheus metrics: https://github.com/doda25-team7/app-service/pull/12

- Tess:  Working to get Istio to function fully and correctly, while also updating various vagrant configuraiton things: https://github.com/doda25-team7/operation/pull/15

### Week Q2.9 (Jan 19 - )
- Maciej: Made the extension proposal: https://github.com/doda25-team7/operation/pull/12
- Francisco - Finishing last week mr, due to conflicts the team reached conclusion that it was better to separate it on a new mr https://github.com/doda25-team7/operation/pull/17. Adding doc and working on final testing.

- Tess: Nasty bug somewhere was causing multiple responses to ARP requests, leading to the join command failing when provisioning. Fixed in https://github.com/doda25-team7/operation/pull/13

- Tobias: https://github.com/doda25-team7/app-service/pull/14
Updated and fixed Active user metric to now use a heartbeat. Made it so host-based kubectl can communicate with the control plane. Made sure that in-class exercises of kubernetes and istio can be deployed and are functional.

### Week Q2.10 (Jan 26 - )
- Tess: Create the plan for the continues deployement: https://github.com/doda25-team7/operation/pull/19
- Francisco: Finished the PR in another file: https://github.com/doda25-team7/operation/pull/17

