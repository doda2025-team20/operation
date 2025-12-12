### Week Q2.1 (Nov 10+)

No work

### Week Q2.2 (Nov 17+)

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
PR Created: https://github.com/doda2025-team20/lib-version/pull/4
PR Approved: https://github.com/doda2025-team20/model-service/pull/7

### Week Q2.3 (Nov 24+)

`Konstantinos Syrros`: https://github.com/doda2025-team20/operation/pull/7\
This week, I worked along with *Norah* on preparing the Infrastructure-as-Code (IaC) for our Kubernetes cluster. Using Vagrant and Ansible, I mainly worked on creating a customizable setup to create a local VM cluster with one controller node and N worker nodes. I also made provisioning configuration that prepares all the nodes with the required setup to install Kubernetes, including disabling swap and proper network configuration. Additionally, I worked more closely with *Norah* on brainstorming and preparing the rest of the Ansible provisioning, to actually install K8s and configure containerd, making the general playbook ready for the others to specialize to the controller and worker nodes.

`Norah E. Milanesi`: https://github.com/doda2025-team20/operation/pull/8\
This week, I worked together with *Konstantinos* on the initial design and setup of our Infrastructure-as-Code workflow for the Kubernetes cluster. We collaborated on the early brainstorming, agreed on the structure of the Ansible provisioning, and aligned on how the controller and worker nodes should be prepared.
After that shared foundation, I focused on implementing some provisioning steps in the general playbook. I added the Kubernetes APT repository configuration, handled the installation of the required Kubernetes tools, and implemented the containerd configuration, including generating the default config and applying the necessary runtime settings. I also set up the `kubelet` service so that it starts now and is enabled on future boots.

`Calin-Stefan Georgescu`: https://github.com/doda2025-team20/operation/pull/6\
This week, I worked on the initial files required for setting up the kubernetes dashboard, as well as istio installation using helm. Additionally, I worked on documenting the steps required to access the kubernetes dashboard securely using kubectl port-forwarding and a bearer token. I also added some sample files for creating an istio gateway. Will provide extra ansible playbooks for installing istio and the dashboard.

'Moegiez Bhatti':
PR Created: https://github.com/doda2025-team20/lib-version/pull/4
PR Approved: https://github.com/doda2025-team20/model-service/pull/7

`Georgi Dimitrov`: https://github.com/doda2025-team20/operation/pull/10
This week I added the initial implementation of the worker nodes's playbook, as well as an initial version of the finalization playbook for MetalLB and Ingres.

### Week Q2.4 (Dec 1+)

`Konstantinos Syrros`: https://github.com/doda2025-team20/operation/pull/14\
This week I took care of migrating our Docker Compose configuration file to one that can be used with Kubernetes on a cluster. I created a configuration with two separate deployments, one for the frontend and one for the backend, allowing them to scale independently. Each deployment has its associated service, and the frontend service is also connected to an ingress. Finally, each deployment has its own ConfigMap, allowing for independent configuration directly through the cluster, and the backend deployment includes a volume mapping for model persistency.

`Norah E. Milanesi`: https://github.com/doda2025-team20/operation/pull/15\
This week, I focused on implementing the Grafana monitoring component. I worked on creating an Helm chart integration for Grafana, which included developing three separate ConfigMaps (for datasources, dashboard provisioning, and the actual dashboards), implementing secret management through values.yaml, and making Grafana dashboard JSON files with multiple visualization types (gauge, timeseries, bar charts), since the other parts were still a work in progress I was not able to test the dashboards and therefore they are still a work in progress. I also wrote the related documentation.

`Georgi Dimitrov`: https://github.com/doda2025-team20/operation/pull/16\
This week, I implemented the alerting requirements for Assignment 3. I introduced the necessary Helm templates to define alerting rules and configure the AlertManager to handle notification via email. The key changes were in the templates/prometheus-rule.yaml (using the sms_requests_total metric from the monitoring part), templates/alertmanager-config.yaml (configuring an AlertmanagerConfig Custom Resource), and templates/alerting-secret.yaml (creating a Kubernetes Secret to store SMTP credentials).

`Calin-Stefan Georgescu`: https://github.com/doda2025-team20/operation/pull/12\
This week I worked on finishing the playbook for the istio and kubernetes dashboard. I helped with the creation of the helm chart, that contains 2 separate deployments ( for both frontend and backend ) as well as matching services to expose the pods and ingress to forward the traffic to the frontend service. I also tested the helm configuration in a different cluster, to make sure it works in a separate envinronment as well. I also reviewed the alerting part, and will review the entire application once everything is merged.


## Week Q2.5 (Dec 8+)

`Norah E. Milanesi`: https://github.com/doda2025-team20/operation/pull/18\
This week, I focused on implementing Istio traffic management. I worked on configuring the Gateway and VirtualServices to route traffic to both v1 and v2 versions of the app-service and model-service, ensuring proper version-specific routing. I also set up DestinationRules and weights for canary deployments and verified sticky sessions for consistent request routing. Additionally, I troubleshot Minikube networking issues, tested service connectivity through the ingress, and documented the setup and troubleshooting steps for Istio integration.

`Konstantinos Syrros`: https://github.com/doda2025-team20/app/pull/8\
Upon Norah's finalization of the Istio traffic management, I implemented a new user interface for the frontend of the application, to represent the canary version to be experimented on. The UI is now comprised of a refreshed experience that is intuitive and modern, with a new color scheme, layout, and design elements. The Helm chart values have been updated (https://github.com/doda2025-team20/operation/pull/19) to include this new version on a 10% traffic weight, while the original version remains at 90%.

`Georgi Dimitrov`: https://github.com/doda2025-team20/operation/pull/21\
This week I went back to our A2 submission, for which we had only an initial attempt at a finalization.yaml playbook. This initial attempt had multiple bugs that prevented the proper setup of MetalLB and Ingress. My PR addressed those bugs, ensuring that MetalLB and Ingress are now properly set up.
