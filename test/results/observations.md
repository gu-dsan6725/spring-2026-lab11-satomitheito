### Observations

### A2A Messages Exchanged Between Agents

During the booking workflow, the Travel Assistant and Flight Booking Agent exchanged messages using the JSON-RPC 2.0 protocol. Here is an example request sent to the Flight Booking Agent:

```json
{
  "jsonrpc": "2.0",
  "id": "test-9a6b9993",
  "method": "message/send",
  "params": {
    "message": {
      "role": "user",
      "parts": [
        {
          "kind": "text",
          "text": "Check availability for flight ID 1"
        }
      ],
      "messageId": "test-msg-e885772b"
    }
  }
}
```

The Flight Booking Agent responded with a result containing artifacts and a history of streamed message chunks:

```json
{
  "id": "test-9a6b9993",
  "jsonrpc": "2.0",
  "result": {
    "artifacts": [
      {
        "artifactId": "78363313-ac66-42c4-9af1-a478a929aee8",
        "name": "agent_response",
        "parts": [
          {
            "kind": "text",
            "text": "Great! Flight ID 1 (UA101) has availability..."
          }
        ]
      }
    ],
    "id": "95a91343-233e-433a-8430-86849c517051",
    "kind": "task",
    "status": {
      "state": "completed",
      "timestamp": "2026-04-07T14:05:39.252863+00:00"
    }
  }
}
```

### How the Travel Assistant Discovered the Flight Booking Agent

The Travel Assistant discovered the Flight Booking Agent through a registry discovery process:

- The Travel Assistant used its discover_remote_agents tool with a natural language query.
- This tool sent a POST request to the Registry Stub with the query string.
- The Registry Stub returned a list of matching agents along with metadata like a relevance score of 0.95 and a trust level of "verified".
- The Travel Assistant cached the discovered agent, making it available for invocation using the invoke_remote_agent tool.

### JSON-RPC Request/Response Format

The A2A protocol uses JSON-RPC 2.0 as its transport format:

Request structure:
- jsonrpc: Always "2.0"
- id: A unique request identifier used to correlate requests with responses
- method: The operation being called message/send for sending a message to an agent
- params.message: Contains role, parts, and a unique messageId

Response structure:
- id: Matches the request id for correlation
- jsonrpc: "2.0"
- result.artifacts: The agent's final response, with an artifactId, name, and parts array
- result.history: A list of streamed message chunks showing how the response was generated incrementally 
- result.status: Task completion status with state and a timestamp
- result.contextId: A session identifier that could be used to continue a conversation

### Agent Card Information and How It Was Used

Each agent exposes an agent card at /.well-known/agent-card.json. The cards contain:

- name/description: Human-readable identity 
- skills: A list of capabilities the agent offers, each with an id, name, and description
- protocolVersion: 0.3.0 
- preferredTransport: JSONRPC
- capabilities: Indicates support for streaming responses
- defaultInputModes/defaultOutputModes: Both set to text
- url: The agent's endpoint for sending messages

The Travel Assistant's agent card also lists discovery-specific skills, which are the tools it uses to find and communicate with other agents. 

### Benefits and Limitations

Benefits:
- Dynamic discovery: The Travel Assistant doesn't need to know about the Flight Booking Agent ahead of time
- Interoperability: The JSON-RPC 2.0 and A2A protocol provide a standard interface, so agents built with different frameworks or languages could communicate as long as they follow the same protocol.
- Composability: Complex workflows can be split across specialized agents, each focused on doing one thing well.

Limitations:
- No real semantic search: The Registry Stub always returns the same agent regardless of the query. A production system would need actual semantic matching to handle diverse queries.
- Scaling challenges: With many agents, every agent needs to discover and communicate with every other agent.
- Latency overhead: A2A communication adds network round-trip time.
- No authentication/authorization: The current setup has no security
