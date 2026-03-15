---
title: "AI Agent Discovery and Invocation Protocol"
abbrev: "AIDIP"
category: info

docname: draft-cui-ai-agent-discovery-invocation
submissiontype: independent
consensus: false
v: 3
area: "Applications and Real-Time"
workgroup: "Independent Submission"
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

As artificial intelligence technologies advance rapidly, AI agents—autonomous software components capable of perceiving their environment, reasoning, and taking actions to achieve goals—have emerged as a powerful paradigm for task execution. Today, many organizations develop specialized AI agents for various purposes, from text translation and summarization to code generation and data analysis. These agents are often offered as services, accessible over the network, and may be integrated into larger systems. However, despite the proliferation of AI agents, there is currently no standard protocol for discovering available agents and invoking their capabilities in a uniform way.

Existing agent frameworks and platforms facilitate building agents but typically operate in isolated ecosystems, making cross-platform or cross-organization agent interoperability difficult. Each platform tends to define its own APIs for agent description and invocation, which means a client wishing to use agents from multiple sources must adapt to disparate interfaces. This lack of standardization creates friction, increases integration costs, and hampers the development of multi-agent collaborative systems.

This document addresses these issues by proposing a standardized AI Agent Discovery and Invocation Protocol. The protocol provides:

1. Agent Metadata Specification: A structured JSON-based description for an agent's identity, capabilities, inputs, outputs, authentication requirements, and other attributes. This enables agents to publish their specifications in a machine-readable form.

2. Discovery Mechanism: A registry-based approach where agents register themselves and clients can search for agents by capability, tags, or semantic queries. The registry is language- and platform-agnostic, facilitating cross-platform discovery.

3. Invocation Interface: A RESTful API that enables a client, which could be a human-facing application, another agent, or an orchestration system, to invoke an agent's capabilities through a standard endpoint and JSON payloads.

4. Security Considerations: Guidelines for authentication, authorization, encrypted transport, and trust establishment, ensuring that discovery and invocation happen securely.

5. Interoperability with Existing Standards: This specification references existing standards such as JSON {{RFC8259}}, HTTP {{RFC9110}}, OAuth 2.0 {{RFC6749}}, and TLS {{RFC8446}}, and leverages established web technologies for broad compatibility.

The primary audience for this specification includes developers of AI agent platforms, providers of AI agent services, and system architects building AI-enabled applications or multi-agent systems. By adopting this protocol, an AI agent developer can make an agent accessible to a wide ecosystem, and a client application can integrate AI agents from multiple vendors without custom integration for each.

This revision extends the base protocol with an optional Agent Semantic Resolution layer that enables intent-based agent selection. This extension allows a Host Agent or coordinator to describe a task intent and receive candidate agents without predetermining which agent to invoke. ASR does not replace discovery; it adds a semantic matching phase that can precede or augment the capability-based search defined in earlier sections.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

- AI Agent: An autonomous software component that can perform tasks using artificial intelligence capabilities. Agents may wrap language models, specialized machine learning models, or reasoning engines, exposing their abilities via defined interfaces.

- Agent Metadata: A structured description of an agent, including name, description, capabilities, input and output schemas, authentication requirements, and endpoint information.

- Agent Registry: A service that maintains a directory of registered agents and supports queries for discovering agents by attributes or semantic search.

- Gateway: An optional intermediary service that routes client requests to appropriate agents.

- Invocation Endpoint: The URL provided by an agent, or by a gateway on behalf of an agent, where clients send invocation requests.

- Capability: A high-level function an agent can perform, identified by a string.

- Operation: A specific action supported by an agent. Some agents may expose multiple operations, each with its own input and output schema.

# Agent Metadata Specification

The Agent Metadata Specification defines a standard JSON document that describes an agent. All agents that wish to be discoverable and invocable through this protocol MUST provide a metadata document conforming to the schema defined by the implementation. This metadata is used for agent registration and returned to clients during discovery.

## Core Fields

The following are the core fields of an agent metadata document:

- id (string): A globally unique identifier for the agent.
- name (string): A human-readable name for the agent.
- description (string): A detailed description of the agent, its purpose, and capabilities in natural language.
- version (string): The version of the agent or its metadata.
- publisher (string): The name or identifier of the entity publishing the agent.
- capabilities (array of strings): A list of high-level capabilities supported by the agent.
- tags (array of strings): Additional tags for search and categorization.
- endpoint (string): The URL of the agent's invocation endpoint.
- supported_languages (array of strings, optional): A list of languages the agent supports.
- authentication (object, optional): A description of the authentication mechanism required to invoke the agent.
- status (string, optional): The operational status of the agent.
- additional fields: Implementations MAY include additional metadata such as rate limits, pricing, or documentation references.

## Operations and I/O Schema

Each agent MUST describe its input and output formats. This is done using the `operations` field:

- operations (array of objects): A list of operations the agent supports. Each operation object includes:
  - name (string): The operation identifier.
  - description (string): A description of what the operation does.
  - inputs (object): A JSON-based schema describing the expected input.
  - outputs (object): A JSON-based schema describing the output format.
  - examples (array of objects, optional): Example input and output pairs.

If an agent has a single operation, this array will have one element. If it supports multiple distinct tasks, each is listed separately. For simple agents with one primary function, an implementation MAY instead use top-level `inputs` and `outputs` fields directly, though use of `operations` is RECOMMENDED for extensibility.

## Example Agent Metadata

Below is an example metadata JSON for a translation agent:

```json
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
    "instructions": "Include 'X-API-Key' header with your API key."
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
