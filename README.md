[![OpenShift Version][openshift39-logo]][openshift39-url]
[![OpenShift Version][openshift310-logo]][openshift310-url]
[![OpenShift Version][openshift311-logo]][openshift311-url]

# OpenShift Examples - Autoscaling
OpenShift can manually or automatically scale application pods up and down based on container metrics such as cpu and memory consumption.  This git repo contains an intentionally simple example of automatically scaling (autoscaling) a webapp frontend with OpenShift.

Here's what it looks like:

![Screenshot](./.screens/ocpautoscale.gif)

###### :information_source: This example is based on OpenShift Container Platform version 3.7.  It could work with older versions but has not been tested.


## How to run this?
First off, you need access to an OpenShift cluster.  Don't have an OpenShift cluster?  That's OK, download the CDK for free here: [https://developers.redhat.com/products/cdk/overview/][5].  Second you have to have [metrics enabled on your cluster][7].

There is a template for creating all the components of this example. Use the oc CLI tool:
 > `oc new-project autoscaledemo `

 > `oc new-app -f https://raw.githubusercontent.com/dudash/openshiftexamples-autoscaling/master/autoscale_instant_template.yaml`

*If you don't like the CLI, another option is to create and project and import the template via the web console:*
 > Create a new project, select `Import YAML/JSON` and then upload the raw file from this repo: `autoscale_instant_template.yaml` and make sure autoscaledemo is set as the Namespace parameter.

Now to showcase the autoscaling - let's simulate a large user load on the frontend webapp using Apache Benchmark.  If you have `ab` installed just run it against the frontend URL.  Or you can use OpenShfit to pull a [container image containing `ab`][6] and run it as self-terminating like this:
 > `oc run web-load --rm --attach --image=jordi/ab -- -n 50000 -c 10 http://URL_GOES_HERE/`


## Why autoscale?
Two of the biggest reasons to leverage this capability in OpenShift:
1) Provide better uptime for your apps when user demand spikes
2) Reduce unecessary compute, scale down during periods of inactivity and up when needed


## How does this work and how can I configure autoscaling?
In order the define autoscaling for an app, we first define how much cpu and memory an instance of the app should consume (both min and max).  This becomes the guideline for OpenShift to know when to scale the pod up or down.  These details are placed into what OpenShift calls a "deployment config" ([you can read more about that here][1]).

In this example if you want to tweak a few things to in the example the following are the most common asks:

Change the min/max number of replicated containers of the web frontend and the CPU request target:
 > `oc autoscale dc/webapp --min 1 --max 5 --cpu-percent=60`

Change the request and limit values for the web frontend deployment:
 > `oc set resources dc/webapp --requests=cpu=200m,memory=256Mi --limits=cpu=400m,memory=512Mi`

You can read more about the details of compute resource request and limits [here][4].  You can read about how Quality of Service (QoS) can be leveraged [here][3].  And read about how to further configure autoscaling [here][2].


## About the code / software architecture
The parts in action here are:
* Simple web front end to collect user input and push to a database or backend API layer
* Backend service, a MongoDB database
* Instant app template YAML file (to create/configure everything easily)
* Key platform components that enable this example
	* container building (source to image)
	* container CPU / memory monitoring
	* container replication and service layer
	* software defined networking and routing layer


## License
Under the terms of the MIT.


[1]: https://docs.openshift.com/container-platform/3.7/architecture/core_concepts/deployments.html#deployments-and-deployment-configurations
[2]: https://docs.openshift.com/container-platform/3.7/dev_guide/pod_autoscaling.html
[3]: https://docs.openshift.com/container-platform/3.7/dev_guide/compute_resources.html#quality-of-service-tiers
[4]: https://docs.openshift.com/container-platform/3.7/dev_guide/compute_resources.html#dev-cpu-requests
[5]: https://developers.redhat.com/products/cdk/overview/
[6]: https://hub.docker.com/r/jordi/ab/
[7]: https://docs.openshift.com/container-platform/3.7/install_config/cluster_metrics.html

[openshift39-url]: https://docs.openshift.com/container-platform/3.9/welcome/index.html
[openshift310-url]: https://docs.openshift.com/container-platform/3.10/welcome/index.html
[openshift311-url]: https://docs.openshift.com/container-platform/3.11/welcome/index.html
[openshift41-url]: https://docs.openshift.com/container-platform/4.1/welcome/index.html

[openshift39-logo]: https://img.shields.io/badge/openshift-3.9-820000.svg?labelColor=grey&logo=data:image/png;base64,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

[openshift310-logo]: https://img.shields.io/badge/openshift-3.10-820000.svg?labelColor=grey&logo=data:image/png;base64,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

[openshift311-logo]: https://img.shields.io/badge/openshift-3.11-820000.svg?labelColor=grey&logo=data:image/png;base64,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

[openshift41-logo]: https://img.shields.io/badge/openshift-4.1-820000.svg?labelColor=grey&logo=data:image/png;base64,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