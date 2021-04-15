Helm chart
==========

This Helm chart can be used to deploy an entire HPCC environment to a K8s cluster.

Add this repository to your helm using:

helm repo add hpcc https://hpcc-systems.github.io/helm-chart

values.yaml sections
--------------------

global:
    # The global section applies to all components within the HPCC system.

dali:
esp:
roxie:
eclccserver:
etc
    # Each section will specify a list of one or more components of the specified type
    # Within each section, there's a map specifying settings specific to that instance of the component,
    # including (at least) name, plus any other required settings (which vary according to component type).

Template structure
------------------

There are some helper templates in _util.tpl to assist in generation of the k8s yaml for each component.
Many of these are used for standard boilerplate that ends up in every component:

hpcc.utils.addImageAttrs
 - adds information about the container image source, version and pull mode
hpcc.utils.addConfigVolumeMount
hpcc.utils.addConfigVolume
 - add information that mount the global and local configuration information into /etc/config using k8s ConfigMap
hpcc.utils.generateConfigMap
 - generates local and global config files for the above
hpcc.utils.configArg
 - generates the parameter to pass to the container naming the config file 
hpcc.utils.daliArg
 - generates the parameter to pass to the container naming the dali to connect to 

Configuration files
-------------------

Each component can specify local configuration via config: or configFile: settings - configFile names a file
that is copied verbatim into the relevant ConfigMap, while config: allows the config file's contents to be
specified inline.

In addition, global config info (same for every component) is generated into a global.json file and made
available via ConfigMap mechanism. So far, this only contains 

  "version": {{ .root.Values.global.image.version | quote }}

but we can add more.

Roxie modes under K8s
---------------------

When running under K8s, roxie has 3 fundamental modes of operation:

  1. Scalable array of one-way Roxie servers

     Set localAgents=true, replicas=initial number of pods

  2. Per-channel-scalable array of combined servers/agents

     localAgent=false, numChannels=nn, replicas=initial number of pods per channel (default 2)

     There will be numChannels*replicas pods in total

  3. Scalable array of servers with per-channel-scalable array of agents

     localAgent=false, numChannels=nn, replicas=pods/channel, serverReplicas=initial number of server pods

     There will be numChannels*replicas agent pods and serverReplicas server pods in total
  
     This mode is somewhat experimental at present!
  
