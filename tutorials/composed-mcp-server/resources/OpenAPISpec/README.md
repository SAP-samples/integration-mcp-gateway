# OpenAPI Specification

This folder contains the OpenAPI Specification that describes the composed `plant-supply-risk` Business Data Graph API. You upload it when creating the MCP Server in [Exercise 2](../../exercises/ex2/README.md#exercise-23--configure-the-mcp-server).

| File | Description |
|---|---|
| `bestrun.assmnt.json` | OpenAPI 3.0.2 spec for the BestRun risk-assessment API. Its operations (under `/bestrun/assmnt`, including `matlStkInAcctMods` for material stock and `locationRiskFactors` for location risk) become the **tools** exposed by the MCP Server. |

> **Usage:** In Exercise 2.3, choose **Upload** and select this file. The MCP Server wizard reads it and lists each operation as a tool the AI agent can call.
