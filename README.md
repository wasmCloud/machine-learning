# machine-learning

This repository shall drive the progress towards a support of ML functionality. The following list summarizes some topics worth to be discussed: 

1. Scope, what to aim for
2. Most important design decisions
3. Rough design of any demo application(s)
4. Target hardware for the demo application
5. Identification of most important risks/open points

## Scope Proposal

1. Inference shall be __IN__ scope
	
> **_[Information]_**  Other actions which are typical for the discipline of machine learning shall deliberately be excluded, e.g. training and data exploration.

## Design decision proposals  

1. A capability provider shall be implemented providing an inference engine (IE).

2. The (first) capability provider shall be implemented based on ([Tract's](https://github.com/sonos/tract/tree/68db0209c9ffd1b91dff82884f4ae03b3622dd34)) [ONNX](https://onnx.ai/) inference engine. Models from nearly all famous ML frameworks can be ported to ONNX format, so the *coverage* should be acceptable from the start.

> **_[Information]_**  [article with design goals of Tract](https://medium.com/snips-ai/snips-open-sources-tract-cdc50f437ef2)

3. An interface shall be implemented designed to support any communication with the IE.

4. The interface shall mimic [WASI-NN](https://github.com/WebAssembly/wasi-nn)

5. There should be a *standard mechanism* how to package and serve AI models. It should be
    - easy to use
    - robust
    - scalable

> **_[Information]_**  An AI model may be served as an actor. An alternative would be to serve an AI model *as-is* (e.g. array of bytes) from a blob store.

## Design proposals for possible example application(s)

### Most basic design

An most basic design of an example application may be illustrated as shown in the following screenshot. It comprises one actor (M) and two capability providers (https and IE). The actor (M) contains the AI model. The first capability provider (https) is an http-server providing an interface to the outside world. The second capability provider (IE) represents the inference engine.

> **_[Note]_**  The advantage of this design is that the appropriate interface closely mimics the one of [WASI-NN](https://github.com/WebAssembly/wasi-nn) with its five functions `load()`, `init_execution_context()`, `set_input()`, 
	`compute()` and `get_output()`. 


<div style="width: 40%; height: 25%">

![most basic design](images/most_basic_setup.png "Most basic design of an example application doing inference")
</div>

<div style="width: 40%; height: 25%">

![legend for most basic design](images/legend_basic_design.png "Most basic design of an example application doing inference")

</div>

#### Data flow

When a request comes in (1) via the http-server (http) it is routed (2) to the actor (M). The actor (M) registers (3) at the inference engine (IE) in case it did not do so before and provides the data (3) to the inference engine (IE). The result is passed (4) from the inference engine (IE) back to the actor (M) which, in turn, passes (5) the result to the http-server (https) where it is sent as a response.

<div style="width: 40%; height: 25%">

![legend for most basic design](images/most_basic_design_data_flow.png "Most basic design of an example application doing inference")

</div>

### More advanced design

A more advanced design additionally introduces an inference client (IC). The inference client (IC) communicates with the http-server (http) and the inference engine (IE) and may be regarded as a *reverse proxy* in that it deals with the opaque handles of the inference engine (IE) alias [GraphExecutionContext](https://github.com/Finfalter/wasmCloudArtefacts/blob/main/interfaces/mlinference/rust/src/mlinference.rs) internally but deals with (nick) names of AI models externally. 

> **_[Note]_**  The use of an inference client (IC) allows an easy deployment of multiple actors (M). However, the interface of such a design would differ from an interface of the basic design.

<div style="width: 40%; height: 25%">

![most basic design](images/more_advanced_design.png "Most basic design of an example application doing inference")
</div>

<div style="width: 40%; height: 25%">

![legend for most basic design](images/legend_more_advanced_design.png "Most basic design of an example application doing inference")

</div>

#### Data flow

The data flow in a more advanced design differs in two important points from the one in the most basic design:

1. If the actor (M) has to register its model at the inference engine (IE) prior to any inference request.
2. Most of the communication is now handled by the inference client (IC). After having registered the actor (M) may as well be removed from the system. 

<div style="width: 40%; height: 25%">

![most basic design](images/more_advanced_design_data_flow_wo_text.png "Most basic design of an example application doing inference")
</div>

The following screenshot illustrates the sequence of a possible inference request. It is done in two steps. A first, optional, step can be to query the available models. The second step is about the inference.

<div style="width: 40%; height: 25%">

![most basic design](images/more_advanced_design_data_flow.png "Most basic design of an example application doing inference")
</div>

### Design with internal data source

It may be advantageous if the data source for inference is internal to the runtime. For example, the internal data source may be represented by a further capability provider (Cam) providing images. In this case it would be necessary to assign this capability provider (Cam) to the AI model of the appropriate actor (M).

<div style="width: 40%; height: 25%">

![most basic design](images/more_advanced_design_internal_data_source.png "Most basic design of an example application doing inference")
</div>

## Target Hardware to select for a demo case

## Risks and Open Points

### [open point] Standard mechanism to package and serve AI models

Current proposals are
1. in form of an actor
2. *as-is* from a blob store, e.g. [bindle](https://github.com/deislabs/bindle), [R2](https://blog.cloudflare.com/introducing-r2-object-storage/), S3 ..

### [open point] Support the Lifecycle of AI models and tensors

Regarding [WASI-NN](https://github.com/WebAssembly/wasi-nn), users of the serving IE may register AI models as well as tensors. However, there seems to be no mechanism yet to unregister such artifacts. For a consideration of any enterprise deployment one whould consider to implement a mechanism to unregister.
