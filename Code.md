# Neo4j Cypher Query Documentation

## Introduction

This document provides detailed explanations of various Neo4j Cypher queries used for analyzing client-server interactions, incidents, and community detection in a networked environment. The queries are designed to work with a specific data model represented in a Neo4j graph database.

## Data Model Overview

The data model for these queries consists of various nodes and relationships. Key nodes include:

1. **Session**: Represents a session in the network.
2. **Incident**: Denotes any incidents occurring within the network.
3. **Server**: A server node in the network.
4. **Client**: A client node interacting with servers.
5. **server_port**: Represents server ports.
6. **client_port**: Represents client ports.

Relationships in the model include `interact`, `categorized`, `interact_through`, and others, signifying different types of interactions and connections between these entities.

## Queries

### Most Frequent Client-Server Interaction

```cypher
MATCH (c:Client)-[r:interact]->(s:Server)
RETURN c, s, COLLECT(r) AS interactions
ORDER BY SIZE(interactions) DESC
```
This query identifies the most frequent interactions between clients and servers. It returns client (`c`) and server (`s`) nodes along with their interactions.

### Most Frequent Client and Client Port

```cypher
MATCH (c:Client)-[r:interact]->()
RETURN c, COLLECT(r) AS interactions, SIZE(COLLECT(r)) AS interaction_count
ORDER BY interaction_count DESC
LIMIT 10
```
This query lists the top 10 clients with the most interactions, providing an insight into the most active clients in the network.

### Most Frequent Server and Server Port

```cypher
MATCH ()-[r:interact]->(sp:server_port)
RETURN sp.destination_port AS server_port, SIZE(COLLECT(r)) AS interaction_count
ORDER BY interaction_count DESC
LIMIT 10
```
Focuses on identifying the most frequently used server ports, highlighting key points of server-client interaction.

### Most Occurring Incidents

```cypher
MATCH ()-[r:categorized]->(i:Incident)
RETURN i, SIZE(COLLECT(r)) AS categorization_count
ORDER BY categorization_count DESC
LIMIT 10
```
Identifies the top 10 incidents based on their frequency of occurrence.

### Louvain Community Detection

```cypher
CALL gds.graph.project(
    'projected_graph',
    ['Session', 'Server', 'Client', 'Incident', 'server_port', 'client_port'],
    {
        categorized: { orientation: 'NATURAL' },
        interact_through: { orientation: 'NATURAL' },
        interact: { orientation: 'NATURAL' },
        request: { orientation: 'NATURAL' },
        response: { orientation: 'NATURAL' }
    }
)

CALL gds.louvain.mutate('projected_graph', { mutateProperty: "louvainCommunityId" })
CALL gds.graph.nodeProperties.write('projected_graph', ["louvainCommunityId"])

MATCH (n)
WHERE n.louvainCommunityId IS NOT NULL
RETURN n.louvainCommunityId AS communityId, COUNT(n) AS countOfNodes
ORDER BY countOfNodes DESC
```
This set of queries is used for community detection using the Louvain method, identifying clusters within the network.

### Largest Community Detection

```cypher
MATCH (client:Client)-[r]->(server:Server)-[p]-(sp:server_port)
WHERE client.louvainCommunityId = largest_id
RETURN client, server, r, p, sp
```
Focuses on identifying members of the largest detected community.

---

