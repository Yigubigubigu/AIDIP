---
title: "AI Agent Discovery and Invocation Protocol"
abbrev: "AIDIP"
category: info

docname: draft-cui-ai-agent-discovery-invocation-latest
submissiontype: independent
consensus: false
v: 3
# area: "Applications and Real-Time"
# workgroup: "Independent Submission"
keyword:
  - AI Agent
  - Service Discovery

author:
  - ins: Y. Cui
    name: Yong Cui
    org: Tsinghua University
    region: Beijing
    code: 100084
    country: China
    email: cuiyong@tsinghua.edu.cn
    uri: http://www.cuiyong.net/

  - ins: Y. Chao
    name: Yihan Chao
    org: Zhongguancun Laboratory
    region: Beijing
    code: 100094
    country: China
    email: chaoyh@zgclab.edu.cn

  - ins: C. Du
    name: Chenguang Du
    org: Zhongguancun Laboratory
    region: Beijing
    code: 100094
    country: China
    email: ducg@zgclab.edu.cn

normative:
  RFC9110:
  RFC8259:
  RFC6749:
  RFC8446:

informative:
  ServiceMesh:
    title: "Service Mesh: The Next Step in Microservices"
    author:
      - name: J. Buisson
    date: 2020
    seriesinfo:
      - name: "Communications of the ACM"
        value: "Vol. 63 No. 12, December 2020"

  LangChain:
    title: "LangChain: Building Applications with LLMs through Composition"
    author:
      - name: H. Chase
    date: 2023
    target: https://www.langchain.com/

  AutoGen:
    title: "AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation"
    author:
      - name: Y. Wu
      - name: et al.
    date: 2023
    seriesinfo:
      - name: "arXiv"
        value: "2308.08155"

  RosenbergDraft:
    title: "AI Protocols"
    author:
      - name: J. Rosenberg
    date: 2023
    seriesinfo:
      - name: "Internet-Draft"
        value: "draft-rosenberg-ai-protocols-00"

--- abstract

This document proposes a standardized protocol for discovery and invocation of AI agents. It defines a common metadata format for describing AI agents, including capabilities, input and output specifications, supported languages, tags, authentication methods, and related attributes. It also defines a capability-based discovery mechanism and a unified RESTful invocation interface.

This revision additionally specifies an optional extension that enables intent-based agent selection prior to discovery and invocation, without changing existing discovery or invocation semantics.

The goal is to enable cross-platform interoperability among AI agents by providing a discover-and-match mechanism and a unified invocation entry point. Security considerations, including authentication and trust measures, are also discussed. This specification aims to facilitate the formation of multi-agent systems by making it easier to find appropriate agents for a task and invoke them consistently across different vendors and platforms.

--- middle


# Introduction

As artificial intelligence technologies advance rapidly, AI agents, autonomous software components capable of perceiving their environment, reasoning, and taking actions to achieve goals, have emerged as a powerful paradigm for task execution. Many organizations now develop specialized AI agents for various purposes, such as text translation, summarization, code generation, and data analysis. These agents are often offered as services accessible over the network and may be integrated into larger systems. However, despite the proliferation of AI agents, there is currently no standard protocol for discovering available agents and invoking their capabilities in a uniform way.

Existing agent frameworks and platforms facilitate building agents but typically operate in isolated ecosystems, making cross-platform or cross-organization agent interoperability difficult. Each platform tends to define its own APIs for agent description and invocation, which means a client wishing to use agents from multiple sources must adapt to disparate interfaces. This lack of standardization creates friction, increases integration costs, and hampers the development of multi-agent collaborative systems.

This document addresses these issues by proposing a standardized AI Agent Discovery and Invocation Protocol. The protocol provides:

1. A metadata specification for describing an agent's identity, capabilities, inputs, outputs, authentication requirements, and other attributes in a machine-readable form.

2. A discovery mechanism that allows agents to register themselves and allows clients to search for agents by capability, tags, supported languages, or semantic queries.

3. An invocation interface that enables a client, which may be a user-facing application, another agent, or an orchestration system, to invoke an agent's capabilities through a standard HTTP endpoint and JSON payload.

4. Security considerations covering authentication, authorization, encrypted transport, and trust establishment to ensure that discovery and invocation happen securely.

5. Interoperability with existing standards, including JSON {{RFC8259}}, HTTP {{RFC9110}}, OAuth 2.0 {{RFC6749}}, and TLS {{RFC8446}}.

The primary audience for this specification includes developers of AI agent platforms, providers of AI agent services, and system architects building AI-enabled applications or multi-agent systems. By adopting this protocol, an AI agent developer can make an agent accessible to a wider ecosystem, and a client application can integrate AI agents from multiple vendors without requiring custom integration for each.

This revision extends the base protocol with an optional Agent Semantic Resolution layer that enables intent-based agent selection. This extension allows a Host Agent or coordinator to describe a task intent and receive candidate agents without predetermining which specific agent to invoke. Agent Semantic Resolution does not replace discovery; it adds a semantic matching phase that can precede or augment the capability-based search defined in earlier sections.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in BCP 14
{{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all
capitals, as shown here.

This document uses the following terms:

AI Agent:
: An autonomous software component that can perform tasks using artificial intelligence capabilities. Agents may wrap language models, specialized machine learning models, or reasoning engines, exposing their abilities via defined interfaces.

Agent Metadata:
: A structured description of an agent, including name, description, capabilities, input and output schemas, authentication requirements, and endpoint information.

Agent Registry:
: A service that maintains a directory of registered agents and supports queries for discovering agents by attributes or semantic search.

Gateway:
: An optional intermediary service that routes client requests to appropriate agents.

Invocation Endpoint:
: The URL provided by an agent, or by a gateway on behalf of an agent, where clients send invocation requests.

Capability:
: A high-level function an agent can perform, identified by a string.

Operation:
: A specific action supported by an agent. Some agents may expose multiple operations, each with its own input and output schema.

# Agent Metadata Specification

The Agent Metadata Specification defines a standard JSON document that describes an agent. All agents that wish to be discoverable and invocable through this protocol MUST provide a metadata document conforming to the applicable schema defined by an implementation of this protocol. This metadata is used for agent registration and returned to clients during discovery.

## Core Fields

The following are the core fields of an agent metadata document:

id:
: A globally unique identifier for the agent.

name:
: A human-readable name for the agent.

description:
: A detailed description of the agent, its purpose, and capabilities in natural language.

version:
: The version of the agent or its metadata.

publisher:
: The name or identifier of the entity publishing the agent.

capabilities:
: A list of high-level capabilities supported by the agent.

tags:
: Additional tags for search and categorization.

endpoint:
: The URL of the agent's invocation endpoint.

supported_languages:
: An optional list of languages the agent supports.

authentication:
: An optional description of the authentication mechanism required to invoke the agent.

status:
: An optional operational status of the agent.

additional fields:
: Implementations MAY include additional metadata such as rate limits, pricing, or documentation references.

## Operations and Input/Output Schema

Each agent MUST describe its input and output formats. This is done using the `operations` field.

operations:
: A list of operations supported by the agent. Each operation includes a `name`, a `description`, an `inputs` object describing the expected input, and an `outputs` object describing the output format. An operation MAY also include example input and output pairs.

If an agent has a single operation, the `operations` array will contain one element. If it supports multiple distinct tasks, each is listed separately. For simple agents with one primary function, an implementation MAY instead use top-level `inputs` and `outputs` fields directly, though use of `operations` is RECOMMENDED for extensibility.

## Example Agent Metadata

Below is an example metadata JSON for a translation agent:

~~~ json
{
  "id": "agent-12345",
  "name": "Chinese-English Translator",
  "description": "Translates text between Chinese and English with high accuracy using a fine-tuned model.",
  "version": "1.2.0",
  "publisher": "ExampleAI Inc.",
  "capabilities": ["translation"],
  "tags": ["nlp", "chinese", "english", "cloud"],
  "endpoint": "https://api.example.com/agents/translate",
  "supported_languages": ["en", "zh"],
  "authentication": {
    "type": "api_key",
    "instructions": "Include X-API-Key header with your API key."
  },
  "status": "active",
  "operations": [
    {
      "name": "translateText",
      "description": "Translates text from source language to target language.",
      "inputs": {
        "type": "object",
        "properties": {
          "text": {"type": "string"},
          "source_language": {"type": "string", "enum": ["en", "zh"]},
          "target_language": {"type": "string", "enum": ["en", "zh"]}
        },
        "required": ["text", "source_language", "target_language"]
      },
      "outputs": {
        "type": "object",
        "properties": {
          "translated_text": {"type": "string"}
        }
      },
      "examples": [
        {
          "input": {"text": "你好世界", "source_language": "zh", "target_language": "en"},
          "output": {"translated_text": "Hello World"}
        }
      ]
    }
  ]
}
~~~

# Agent Discovery Mechanism

The discovery mechanism allows clients to find agents that meet certain criteria. Discovery is provided by an Agent Registry that aggregates metadata from multiple agents.

## Registry Overview

The Agent Registry is a network-accessible service that:

1. Allows agents or their administrators to register metadata about the agent.

2. Stores and indexes these metadata entries for efficient search.

3. Provides endpoints for clients to query and retrieve agent information.

A registry may be operated by an organization for its internal agents, or by a third party acting as a directory of agents across multiple providers. Multiple registries can coexist. Interoperability between registries is facilitated by consistent metadata formats, though formal registry federation is out of scope for this document.

## Agent Registration

An agent, or its administrator, registers with the registry by sending metadata to a registration endpoint.

Endpoint:
: `POST /agents`

Request Body:
: The agent metadata JSON document.

Response:
: On success, the registry returns `201 Created` for a new agent or `200 OK` for an updated entry, together with the stored metadata.

The registry MUST validate the metadata against the applicable schema. Registration MAY require authentication.

Updates to an agent's metadata can be performed using:

Endpoint:
: `PUT /agents/{id}`

Request Body:
: Updated metadata.

Response:
: `200 OK` on success, `404 Not Found` if the agent does not exist, or `403 Forbidden` if the requester is not authorized.

## Querying Agents

Clients query the registry using a search endpoint.

### Attribute-Based Query

Clients can specify criteria to filter agents by capabilities, tags, supported languages, and related attributes.

Example request body:

~~~ json
{
  "filters": {
    "capabilities": ["translation"],
    "supported_languages": ["en", "zh"],
    "tags": ["nlp"]
  },
  "top": 10
}
~~~

A registry MAY support either a GET-based query form or a structured `POST /agents/search` form. For more complex queries, `POST /agents/search` is RECOMMENDED.

The response is a JSON array of agent summary objects containing enough information for a client to select a candidate agent.

### Semantic Query

In addition to attribute-based search, the registry MAY support semantic search in which the client describes a need in natural language and the registry uses semantic matching techniques to identify relevant agents.

Example request body:

~~~ json
{
  "query": "I need an agent that can summarize long legal documents in Chinese.",
  "top": 5
}
~~~

The registry returns a ranked list of candidate agents. Registries that do not support semantic search MAY ignore the `query` field and rely on available filters only.

### Retrieve Single Agent

Endpoint:
: `GET /agents/{id}`

Response:
: Full metadata for the specified agent, or `404 Not Found`.

# Agent Invocation

Once a client discovers a suitable agent, it invokes the agent by sending a request to the agent's endpoint.

## Invocation Request

To invoke an agent, the client sends an HTTP `POST` request to the agent's invocation endpoint with a JSON body containing input data for the task.

Method:
: `POST`

URL:
: The `endpoint` value from the agent's metadata.

Headers:
: `Content-Type: application/json`, plus authentication headers as required.

Body:
: A JSON object containing input data as defined by the agent's schema.

Example request body:

~~~ json
{
  "text": "Hello, how are you?",
  "source_language": "en",
  "target_language": "fr"
}
~~~

If the agent supports multiple operations through a unified endpoint, the request MAY include an operation selector such as `"operation": "translateText"`.

## Invocation Response

The agent or gateway processes the request and returns a response.

A successful invocation SHOULD return `200 OK` with a JSON body conforming to the advertised output schema.

A malformed or invalid request SHOULD return an appropriate 4xx response together with an error object. For example:

~~~ json
{
  "error": {
    "code": "InvalidInput",
    "message": "Required field target_language is missing."
  }
}
~~~

An agent-side processing failure SHOULD return an appropriate 5xx response together with an error object.

## Additional Considerations for Invocation

Streaming responses:
: Implementations MAY support streaming extensions for incremental outputs.

Batch requests:
: Implementations MAY support batch invocation.

Idempotency and retries:
: Clients and gateways should use retry logic carefully because some operations may have side effects.

Operation metadata:
: Multiple operations MAY be exposed through a generic endpoint or through separate endpoints.

# Agent Semantic Resolution

Agent Semantic Resolution, or ASR, is an optional extension to the discovery mechanism defined in this document. ASR enables a client to resolve a task intent into one or more candidate agents prior to invoking any specific agent.

ASR operates on the following conceptual model:

~~~ text
(Intent, Context, Policy) -> (Agent Endpoint(s), Invocation Metadata)
~~~

The intent represents the task to be performed, while context and policy may include domain constraints, trust requirements, or performance considerations.

## Non-Goals

ASR does not provide:

- name-to-address resolution,
- global or persistent agent identifiers, or
- replacement for DNS, ANS, or URI-based registries.

ASR answers the question "Which agent should handle this task now?" rather than "Where is agent X located?".

## Semantic Routing Platform

A Semantic Routing Platform is a control-plane service that implements ASR. It assists a Host Agent in selecting candidate agents before standard discovery and invocation procedures are used.

A Semantic Routing Platform MAY perform semantic matching, ranking, and policy-based filtering of candidate agents. It does not participate in task execution and does not alter the invocation semantics defined in this document.

Interaction with such a platform is OPTIONAL. Clients that do not support ASR continue to operate using the discovery and invocation mechanisms defined in earlier sections.

## Backward Compatibility

All discovery and invocation mechanisms defined in previous revisions of this document remain valid and unchanged.

ASR is an optional extension. Implementations MAY support ASR incrementally, and registries MAY provide semantic resolution capabilities without affecting existing clients.

# Security Considerations

Security is a critical aspect of this protocol. All discovery and invocation traffic MUST be protected with TLS {{RFC8446}}. Authentication mechanisms such as OAuth 2.0 bearer tokens {{RFC6749}}, API keys, or mutual TLS are required except for public discovery endpoints.

Registries MUST enforce per-client entitlements so that both search results and invocation access respect permissions and scopes. Gateways forwarding requests SHOULD authenticate themselves to agents. Agents SHOULD maintain stable identifiers and MAY use signed responses when integrity protection is required.

To mitigate abuse, registries and agents MUST implement rate limiting and quotas, particularly in semantic search scenarios. Trust mechanisms such as certification, testing, or reputation systems MAY be used to validate agent claims, and metadata fields such as certification or quality indicators MAY assist client trust decisions.

Systems SHOULD provide audit and logging with privacy-aware retention. Clients MUST treat agent outputs as untrusted until verified and SHOULD apply sandboxing or validation before executing returned code or commands.

When ASR is used, security considerations extend to the pre-invocation phase. Resolution services SHOULD validate agent capability claims, apply policy constraints, and exclude agents that do not meet trust or reputation requirements.

# IANA Considerations

This document has no IANA actions.

# Acknowledgments

{:numbered="false"}

The authors thank the contributors and reviewers who provided comments on earlier versions of this document.

--- back
