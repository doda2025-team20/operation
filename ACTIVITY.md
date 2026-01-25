### Week Q2.1 (Nov 10+)

No activity

### Week Q2.2 (Nov 17 - Nov 23)

`Norah E. Milanesi`: https://github.com/doda2025-team20/app/pull/1\
During this week I collaborated with *Konstantinos* and together we worked on containerizing both frontend and backend, supporting multiple architectures, stages and making them flexible.

`Georgi Dimitrov`: https://github.com/doda2025-team20/lib-version/pull/2\
During this week I built on top of the initial implementation of the `lib-version` library done by *Petre*. I extended it by automating the release process using a release workflow, as well as changing the versioning file from a version.txt to a version.properties file that automatically fetches the version from the Maven build metadata.

`Calin-Stefan Georgescu`: [app]https://github.com/doda2025-team20/app/pull/3 [model-service]https://github.com/doda2025-team20/model-service/pull/7.
This week, I worked on creating an automated workflow for building multi arch containers for both the app and model service. I started by creating a separate repostory for common tools that can be reused: https://github.com/doda2025-team20/build-tools. I created a workflow that can build and push the container, and then called that workflow from both the app and model service. 

`Konstantinos Syrros`: https://github.com/doda2025-team20/model-service/pull/5\
This week, I worked on containerizing the `model-service` backend. I created the initial multi-stage Dockerfile that, based on the instructions on running the service locally, trains the model and starts the Python Flask server to serve it. Later on, I added the functionality to source the model externally from a mounted volume or from a GitHub release, to avoid retraining the model on every container build, which was time-consuming especially when cross-building. I also created a workflow to automatically train the model and create an artifact, when its source code changes. Finally, I created and documented the `docker-compose.yml` file to orchestrate the `app` and `model-service` containers together, in the `operation` repository.

`Petre-Alexandru Hautelman`: [lib-version](https://github.com/doda2025-team20/lib-version/pull/1), [app](https://github.com/doda2025-team20/app/pull/4/files)\
I worked on the initial implementation of the `lib-version` library. Additionally, I focused on installing and integrating the library into the `app` project. This involved making modifications to the `app` Dockerfile and the GitHub workflow, such that the Maven m2 secrets are not included in the published Docker image.

`Moegiez Bhatti`: 

**PRs Created:**

https://github.com/doda2025-team20/lib-version/pull/4


**PRs Approved:**

https://github.com/doda2025-team20/model-service/pull/7


### Week Q2.3 (Nov 24 - Nov 30)

`Konstantinos Syrros`: https://github.com/doda2025-team20/operation/pull/7\
This week, I worked along with *Norah* on preparing the Infrastructure-as-Code (IaC) for our Kubernetes cluster. Using Vagrant and Ansible, I mainly worked on creating a customizable setup to create a local VM cluster with one controller node and N worker nodes. I also made provisioning configuration that prepares all the nodes with the required setup to install Kubernetes, including disabling swap and proper network configuration. Additionally, I worked more closely with *Norah* on brainstorming and preparing the rest of the Ansible provisioning, to actually install K8s and configure containerd, making the general playbook ready for the others to specialize to the controller and worker nodes.

`Norah E. Milanesi`: https://github.com/doda2025-team20/operation/pull/8\
This week, I worked together with *Konstantinos* on the initial design and setup of our Infrastructure-as-Code workflow for the Kubernetes cluster. We collaborated on the early brainstorming, agreed on the structure of the Ansible provisioning, and aligned on how the controller and worker nodes should be prepared.
After that shared foundation, I focused on implementing some provisioning steps in the general playbook. I added the Kubernetes APT repository configuration, handled the installation of the required Kubernetes tools, and implemented the containerd configuration, including generating the default config and applying the necessary runtime settings. I also set up the `kubelet` service so that it starts now and is enabled on future boots.

`Calin-Stefan Georgescu`: https://github.com/doda2025-team20/operation/pull/6\
This week, I worked on the initial files required for setting up the kubernetes dashboard, as well as istio installation using helm. Additionally, I worked on documenting the steps required to access the kubernetes dashboard securely using kubectl port-forwarding and a bearer token. I also added some sample files for creating an istio gateway. Will provide extra ansible playbooks for installing istio and the dashboard.

`Moegiez Bhatti`: 

**PRs Created:**

https://github.com/doda2025-team20/model-service/pull/10 (Extra commit w2, compensation for w3)

**PRs Approved:**

TBD (extra approval in w10)

`Georgi Dimitrov`: https://github.com/doda2025-team20/operation/pull/10
This week I added the initial implementation of the worker nodes's playbook, as well as an initial version of the finalization playbook for MetalLB and Ingres.

`Petre-Alexandru Hautelman`: https://github.com/doda2025-team20/operation/pull/9
Worked together with Moegiez to implement the Kubernetes controller playbook, steps 13-17 of the assignment.

### Week Q2.4 (Dec 1 - Dec 7)

`Konstantinos Syrros`: https://github.com/doda2025-team20/operation/pull/14\
This week I took care of migrating our Docker Compose configuration file to one that can be used with Kubernetes on a cluster. I created a configuration with two separate deployments, one for the frontend and one for the backend, allowing them to scale independently. Each deployment has its associated service, and the frontend service is also connected to an ingress. Finally, each deployment has its own ConfigMap, allowing for independent configuration directly through the cluster, and the backend deployment includes a volume mapping for model persistency.

`Norah E. Milanesi`: https://github.com/doda2025-team20/operation/pull/15\
This week, I focused on implementing the Grafana monitoring component. I worked on creating an Helm chart integration for Grafana, which included developing three separate ConfigMaps (for datasources, dashboard provisioning, and the actual dashboards), implementing secret management through values.yaml, and making Grafana dashboard JSON files with multiple visualization types (gauge, timeseries, bar charts), since the other parts were still a work in progress I was not able to test the dashboards and therefore they are still a work in progress. I also wrote the related documentation.

`Georgi Dimitrov`: https://github.com/doda2025-team20/operation/pull/16\
This week, I implemented the alerting requirements for Assignment 3. I introduced the necessary Helm templates to define alerting rules and configure the AlertManager to handle notification via email. The key changes were in the templates/prometheus-rule.yaml (using the sms_requests_total metric from the monitoring part), templates/alertmanager-config.yaml (configuring an AlertmanagerConfig Custom Resource), and templates/alerting-secret.yaml (creating a Kubernetes Secret to store SMTP credentials).

`Calin-Stefan Georgescu`: https://github.com/doda2025-team20/operation/pull/12\
This week I worked on finishing the playbook for the istio and kubernetes dashboard. I helped with the creation of the helm chart, that contains 2 separate deployments ( for both frontend and backend ) as well as matching services to expose the pods and ingress to forward the traffic to the frontend service. I also tested the helm configuration in a different cluster, to make sure it works in a separate envinronment as well. I also reviewed the alerting part, and will review the entire application once everything is merged.

`Petre-Alexandru Hautelman`: https://github.com/doda2025-team20/operation/pull/12
I worked on the initial migration of the Docker Compose files to Kubernetes. I created a Helm chart for deploying the model and application services, which connects the app to the model server. The deployments utilize configurable variables for the name, image, and port, and the model service includes an optional shared folder.

`Moegiez Bhatti`: TO BE SPECIFIED

**PRs Created:**

https://github.com/doda2025-team20/app/pull/7

**PRs Approved:**

https://github.com/doda2025-team20/operation/pull/15

### Week Q2.5 (Dec 8 - 14)

`Norah E. Milanesi`: https://github.com/doda2025-team20/operation/pull/18\
This week, I focused on implementing Istio traffic management. I worked on configuring the Gateway and VirtualServices to route traffic to both v1 and v2 versions of the app-service and model-service, ensuring proper version-specific routing. I also set up DestinationRules and weights for canary deployments and verified sticky sessions for consistent request routing. Additionally, I troubleshot Minikube networking issues, tested service connectivity through the ingress, and documented the setup and troubleshooting steps for Istio integration.

`Konstantinos Syrros`: https://github.com/doda2025-team20/app/pull/8\
Upon Norah's finalization of the Istio traffic management, I implemented a new user interface for the frontend of the application, to represent the canary version to be experimented on. The UI is now comprised of a refreshed experience that is intuitive and modern, with a new color scheme, layout, and design elements. The Helm chart values have been updated (https://github.com/doda2025-team20/operation/pull/19) to include this new version on a 10% traffic weight, while the original version remains at 90%.

`Georgi Dimitrov`: https://github.com/doda2025-team20/operation/pull/21\
This week I went back to our A2 submission, for which we had only an initial attempt at a finalization.yaml playbook. This initial attempt had multiple bugs that prevented the proper setup of MetalLB and Ingress. My PR addressed those bugs, ensuring that MetalLB and Ingress are now properly set up.

`Calin-Stefan Georgescu`: https://github.com/doda2025-team20/operation/pull/20\
This week, I worked on implementing a shadow launch strategy for the model service using Istio. I modified the existing VirtualService configuration to include traffic mirroring from the stable version (v1) to the newer shadown version (v3) of the model service. This setup allows us to test the new version under real traffic conditions without impacting the user experience. Additionally, I worked on writing documentation for our entire deployment strategies, including both canary deployment and shadow launch.

`Moegiez Bhatti`: TO BE SPECIFIED

**PRs Created:**

https://github.com/doda2025-team20/model-service/pull/11

**PRs Approved:**

https://github.com/doda2025-team20/operation/pull/20

https://github.com/doda2025-team20/operation/pull/18

### Week Q2.6 (Dec 15 - Dec 21)

`Norah E. Milanesi`: https://github.com/doda2025-team20/operation/pull/25\
This week I worked on adding full monitoring support for the application by integrating Prometheus metrics with Grafana dashboards. The dashboards were configured to display the required metrics using multiple visualizations, including gauges, time series, bar charts, and pie charts, covering request rates, classification counts, confidence scores, and latency. Additional panels were created to clearly show traffic distribution and behavior between the stable (v1) and canary (v2) versions. To ensure accurate latency calculations, the MetricsController in the app repository was updated to expose cumulative histogram buckets, allowing Prometheus and Grafana to aggregate and display the metrics correctly as traffic is generated.

`Moegiez Bhatti`: TO BE SPECIFIED

**PRs Created:**

https://github.com/doda2025-team20/operation/pull/24

**PRs Approved:**

(https://github.com/doda2025-team20/operation/pull/25)

https://github.com/doda2025-team20/app/pull/11


### Week Q2.7 (Jan 5 - Jan 11)

`Konstantinos Syrros`: https://github.com/doda2025-team20/operation/pull/26\
Worked on properly reimplementing the sticky session routing using a `canary` cookie in Istio, as the previous implementation was for pod-level sticky sessions and did not work as intended for sticky versions. The new implementation ensures that users are consistently routed to the same version of the app-service based on the `canary` cookie, enhancing the user experience during canary deployments. For the time being, this is to be tested by manually setting the cookie in the browser, however implementation of automatic cookie setting in the frontend is in the works.

`Georgi Dimitrov`: https://github.com/doda2025-team20/operation/pull/29\
This week I changed the Vagrantfile and the ansible playbooks so that the /etc/hosts file in each node is dynamically generated. Beforehand we had a hardcoded /infra/playbooks/hosts file that was manually copied into each of the nodes, and now we use ansible.builtin.blockinfile to create the /etc/hosts file dynamically instead.

`Moegiez Bhatti`: TO BE SPECIFIED

**PRs Created:**

https://github.com/doda2025-team20/operation/pull/28

**PRs Approved:**

(https://github.com/doda2025-team20/operation/pull/27)
### Week Q2.8 (Jan 12 - 18)

`Konstantinos Syrros`: https://github.com/doda2025-team20/operation/pull/32\
This week I worked on thoroughly testing and reconfiguring the AlertManager setup, which was not properly functioning before. Alerts configured in Prometheus Rules would not reach the email receiver due to misconfigurations in the matchers and routing tree, leading them to always reach the default `null` receiver. Instead, Kubernetes cluster alerts were only being routed. The routing tree was restructured to ensure all alerts from Prometheus Rules in the release namespace (`default`) reach the email receiver. Further, I rewrote the Helm template for the AlertManager configuration to improve readability and maintainability, and use the values from `values.yml` properly. Finally, I tested the entire alerting flow by creating a test alert in Prometheus Rules and verifying that it was received via email.

`Georgi Dimitrov`: https://github.com/doda2025-team20/operation/pull/35\
This week I identified andfixed a shadow launch issue. While in our values.yaml we had already defined shadow.enables, shadow.image, shadow.tag, shadow.mirrorPercentage, we had an undefined outlierDetection.enabled, which was causing an error. Furthermore, the model service deployment was not deploying the shadow version, and therefore the shadow pod didn't exist to receive mirrored traffic. Finally, I added the traffic mirroring configuration to the model service virtual service.

`Norah E. Milanesi`:https://github.com/doda2025-team20/operation/pull/37
This week, I improved the project README based on peer feedback, adding clear instructions for accessing the application via Istio Ingress, linking detailed READMEs to avoid duplication, and clarifying the roles of Helm, Prometheus, and Grafana. I also updated repository links and reorganized the content for a clearer flow from local deployment to Kubernetes monitoring. All changes were documentation-only.

`Calin-Stefan Georgescu`: https://github.com/doda2025-team20/operation/pull/31 && https://github.com/doda2025-team20/operation/pull/34
This week, I worked on documenting extension proposal, and also provide an implementation for it. The extension consists of using GitOps for managing the cluster configuration, as well as automating the deployment of the application using FluxCD. I created a sample repository that contains the helm chart, as well as the configuration required for deploying the application using FluxCD. I also wrote documentation that explains why this extension was chosen. I also added a pipeline that automatically releases a new version of the application helm chart, everytime there is a new change in the chart repository.

`Moegiez Bhatti`: TO BE SPECIFIED

**PRs Created:**

https://github.com/doda2025-team20/operation/pull/36

**PRs Approved:**

https://github.com/doda2025-team20/operation/pull/32



### Week Q2.9 (Jan 19 - 25)

`Norah E. Milanesi`: https://github.com/doda2025-team20/model-service/pull/13\
This week I improved the SMS spam detection system's observability and metrics accuracy by adding confidence score calculation to the model service to return actual prediction probabilities instead of placeholder values. I also updated the frontend to integrate these dynamic confidence scores and added version labels to all Prometheus metrics for better tracking across deployments.

`Georgi Dimitrov`: https://github.com/doda2025-team20/operation/pull/39\
This week I worked on documentation mainly for assignment 4. I looked at our docs/deployment.md file and fixed some inconsistencies regarding the shadow launch deployment. I also described our usage of sticky sessions for consistent user experience, as well as the option to manually set a canary cookie as a developer in order to use another deployment of the application. In our main README.md, I included information about our extension - GitOps with Flux. Finally, I explained our deployments, service mesh, and traffic management using Istio, including implementation and architecture.

`Konstantinos Syrros`: https://github.com/doda2025-team20/operation/pull/40\
This week I reworked the Prometheus stack installation, which before was happening manually and on a separate namespace, in a non-standardized way. `kube-prometheus-stack` has now been included as a dependency in our Helm Chart, and the `README.md` has been updated with the new installation and usage instructions. Prometheus and AlertManager configurations have been updated to be properly targeted by the operator, and Grafana has been properly reconfigured to use the one deployed by the stack, instead of deploying our own version manually. Grafana is now accessible through either Ingress or Istio VirtualService at `/grafana`.

`Calin-Stefan Georgescu`: https://github.com/doda2025-team20/team20-flux/pull/1\
This week I debugged the new workflow and component that automatically release a new version of the helm chart any time a change happens. The wrong token was used when the job was run, and the package location settings had to be changed to allow this current repo to push a new chart. I also spent time reviewing the final documentations and make sure the commands mentioned also work on different Operating Systems. I also wrote documentation for the flux repo used, providing information on the structure, usage and useful commands for debugging it.
