 # A2A Protocol Demos

This document contains official demos and examples for the Agent2Agent (A2A) Protocol.

## Basic Examples

### 1. Simple Agent Communication

```python
from a2a import Agent, Message

# Create two agents
agent1 = Agent("agent1")
agent2 = Agent("agent2")

# Define message handler
@agent1.on_message
async def handle_message(message: Message):
    print(f"Agent1 received: {message.content}")
    # Send response back
    await agent1.send_message(
        Message(
            to="agent2",
            content="Hello from Agent1!",
            type="response"
        )
    )

# Start communication
async def main():
    # Send initial message
    await agent2.send_message(
        Message(
            to="agent1",
            content="Hello from Agent2!",
            type="request"
        )
    )
    
    # Keep the agents running
    await asyncio.gather(agent1.run(), agent2.run())

if __name__ == "__main__":
    asyncio.run(main())
```

### 2. Multi-Agent Task Coordination

```python
from a2a import Agent, Message, Task

# Create specialized agents
coordinator = Agent("coordinator")
researcher = Agent("researcher")
analyzer = Agent("analyzer")

# Define task handlers
@coordinator.on_task
async def coordinate_research(task: Task):
    # Split task into subtasks
    research_task = Task(
        id=f"{task.id}_research",
        description="Research market trends",
        data=task.data
    )
    
    analysis_task = Task(
        id=f"{task.id}_analysis",
        description="Analyze research results",
        data=task.data
    )
    
    # Assign tasks to specialized agents
    await coordinator.assign_task(research_task, researcher)
    await coordinator.assign_task(analysis_task, analyzer)

@researcher.on_task
async def perform_research(task: Task):
    # Perform research
    results = await research_market(task.data)
    # Send results to coordinator
    await researcher.send_message(
        Message(
            to="coordinator",
            content=results,
            type="research_results",
            task_id=task.id
        )
    )

@analyzer.on_task
async def analyze_results(task: Task):
    # Analyze research results
    analysis = await analyze_data(task.data)
    # Send analysis to coordinator
    await analyzer.send_message(
        Message(
            to="coordinator",
            content=analysis,
            type="analysis_results",
            task_id=task.id
        )
    )
```

### 3. State Management Example

```python
from a2a import Agent, Message, State

# Create stateful agents
class StatefulAgent(Agent):
    def __init__(self, name: str):
        super().__init__(name)
        self.state = State()
    
    async def update_state(self, key: str, value: any):
        await self.state.set(key, value)
        # Notify other agents about state change
        await self.broadcast_message(
            Message(
                content={"key": key, "value": value},
                type="state_update"
            )
        )

# Create agents
agent1 = StatefulAgent("agent1")
agent2 = StatefulAgent("agent2")

# Handle state updates
@agent1.on_message
async def handle_state_update(message: Message):
    if message.type == "state_update":
        key = message.content["key"]
        value = message.content["value"]
        await agent1.update_state(key, value)
```

## Advanced Examples

### 1. Secure Communication

```python
from a2a import Agent, Message, Security

# Create secure agents
agent1 = Agent("agent1", security=Security(
    encryption=True,
    authentication=True
))
agent2 = Agent("agent2", security=Security(
    encryption=True,
    authentication=True
))

# Secure message exchange
async def secure_communication():
    # Send encrypted message
    await agent1.send_message(
        Message(
            to="agent2",
            content="Secure message",
            type="secure",
            encryption="aes-256-gcm"
        )
    )
```

### 2. Streaming Data Exchange

```python
from a2a import Agent, Message, Stream

# Create streaming agents
class StreamingAgent(Agent):
    def __init__(self, name: str):
        super().__init__(name)
        self.stream = Stream()
    
    async def process_stream(self, data: bytes):
        # Process streaming data
        processed = await self.stream.process(data)
        return processed

# Create agents
agent1 = StreamingAgent("agent1")
agent2 = StreamingAgent("agent2")

# Handle streaming data
@agent1.on_stream
async def handle_stream(stream: Stream):
    async for data in stream:
        processed = await agent1.process_stream(data)
        await agent1.send_message(
            Message(
                to="agent2",
                content=processed,
                type="stream_data"
            )
        )
```

### 3. Error Handling and Recovery

```python
from a2a import Agent, Message, Error

# Create agents with error handling
class RobustAgent(Agent):
    async def handle_error(self, error: Error):
        # Log error
        print(f"Error occurred: {error.message}")
        
        # Attempt recovery
        if error.type == "connection":
            await self.reconnect()
        elif error.type == "state":
            await self.state.recover()
        
        # Notify other agents
        await self.broadcast_message(
            Message(
                content={"error": error.message, "recovery": "attempted"},
                type="error_notification"
            )
        )

# Create robust agents
agent1 = RobustAgent("agent1")
agent2 = RobustAgent("agent2")

# Error handling
@agent1.on_error
async def handle_agent_error(error: Error):
    await agent1.handle_error(error)
```

## Best Practices

1. **Error Handling**
   - Always implement error handlers
   - Use appropriate error types
   - Implement recovery mechanisms

2. **State Management**
   - Keep state updates atomic
   - Implement state recovery
   - Use appropriate state storage

3. **Security**
   - Enable encryption for sensitive data
   - Implement authentication
   - Use secure message types

4. **Performance**
   - Use streaming for large data
   - Implement proper cleanup
   - Monitor resource usage

## Additional Resources

- [Official A2A Documentation](https://ai.google.dev/docs/a2a_protocol)
- [A2A GitHub Repository](https://github.com/google/a2a-protocol)
- [A2A Examples Repository](https://github.com/google/a2a-protocol/examples)