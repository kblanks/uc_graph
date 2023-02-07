//Creating this document to remember my process for creating, loading, and verifying this data
//Zrzl code in CDE in ^X19798UCGRAPH
//Copy-paste screen output to Excel and save three separate .csv files
//Upload csv files to Github 
//Use the listed Cypher queries to load the data into Neo4j database
//Sample queries of the data at the bottom

//DATA LOAD 
LOAD CSV WITH HEADERS FROM "https://raw.githubusercontent.com/kblanks/uc_graph/main/tlk.csv" as tlk
merge (conv:Conversation {convID: tlk.TLK_ID})
    on create set conv.convName = tlk.CONV_TITLE

MATCH (c:Conversation) return c;

LOAD CSV WITH HEADERS FROM "https://raw.githubusercontent.com/kblanks/uc_graph/main/emp.csv" as emp
merge (user:User {userID: emp.EMP_ID})
    on create set user.userName = emp.EMP_NAME

MATCH (u:User) return u;

CREATE CONSTRAINT conversation_id FOR (c:Conversation) REQUIRE c.convID IS UNIQUE;
CREATE CONSTRAINT user_id FOR (u:User) REQUIRE u.userID IS UNIQUE;
CALL db.awaitIndexes();

LOAD CSV WITH HEADERS FROM "https://raw.githubusercontent.com/kblanks/uc_graph/main/tlk_emp.csv" as row
MATCH (conv:Conversation {convID: row.TLK_ID})
MATCH (user:User {userID: row.EMP_ID})
MERGE (conv)-[op:CONTAINS]->(user)

MATCH (c:Conversation)-[]-(u:User)
RETURN c,u LIMIT 10;

//SAMPLE QUERIES
//All conversations for a single user
MATCH (c:Conversation)-[]-(u:User {userID:'BCOE2'})
RETURN c,u

//Top 10 users by conversation count
MATCH (c:Conversation)-[r]-(u:User)
RETURN u.userID,count(r) as Conv_Count
ORDER BY Conv_Count DESC
LIMIT 10;

//All related nodes for Conversations involving a specific user
MATCH (c:Conversation)-[]-(u:User {userID:'JCASTE12'})
WITH c
MATCH (c)--(u2)
RETURN c,u2
