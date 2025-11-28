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

### Week Q2.3 (Nov 24+)

`Konstantinos Syrros`: https://github.com/doda2025-team20/operation/pull/7\
This week, I worked along with *Norah* on preparing the Infrastructure-as-Code (IaC) for our Kubernetes cluster. Using Vagrant and Ansible, I mainly worked on creating a customizable setup to create a local VM cluster with one controller node and N worker nodes. I also made provisioning configuration that prepares all the nodes with the required setup to install Kubernetes, including disabling swap and proper network configuration. Additionally, I worked more closely with *Norah* on brainstorming and preparing the rest of the Ansible provisioning, to actually install K8s and configure containerd, making the general playbook ready for the others to specialize to the controller and worker nodes.

`Norah E. Milanesi`: https://github.com/doda2025-team20/operation/pull/8\
This week, I worked together with *Konstantinos* on the initial design and setup of our Infrastructure-as-Code workflow for the Kubernetes cluster. We collaborated on the early brainstorming, agreed on the structure of the Ansible provisioning, and aligned on how the controller and worker nodes should be prepared.
After that shared foundation, I focused on implementing some provisioning steps in the general playbook. I added the Kubernetes APT repository configuration, handled the installation of the required Kubernetes tools, and implemented the containerd configuration, including generating the default config and applying the necessary runtime settings. I also set up the `kubelet` service so that it starts now and is enabled on future boots.

`Calin-Stefan Georgescu`: https://github.com/doda2025-team20/operation/pull/6\
This week, I worked on the initial files required for setting up the kubernetes dashboard, as well as istio installation using hel,. Additionally, I worked on documenting the steps required to access the kubernetes dashboard securely using kubectl port-forwarding and a bearer token. I also added some sample files for creating an istio gateway. Will provide extra ansible playbooks for installing istio and the dashboard.

`Moegiez Bhatti`:
PR Created: https://github.com/doda2025-team20/lib-version/pull/4
PR Approved: https://github.com/doda2025-team20/model-service/pull/7

`Georgi Dimitrov`: https://github.com/doda2025-team20/operation/pull/10
This week I added the initial implementation of the worker nodes's playbook, as well as an initial version of the finalization playbook for MetalLB and Ingres.
