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
