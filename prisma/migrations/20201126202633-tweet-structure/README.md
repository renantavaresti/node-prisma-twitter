# Migration `20201126202633-tweet-structure`

This migration has been generated by Diego Fernandes at 11/26/2020, 5:26:33 PM.
You can check out the [state of the schema](./schema.prisma) after the migration.

## Database Steps

```sql
ALTER TABLE "User" DROP CONSTRAINT "User_followingId_fkey"

ALTER TABLE "User" DROP CONSTRAINT "User_pkey",
DROP COLUMN "followingId",
ADD COLUMN     "password" TEXT NOT NULL,
ADD COLUMN     "userId" TEXT,
ALTER COLUMN "id" DROP DEFAULT,
ALTER COLUMN "id" SET DATA TYPE TEXT,
ADD PRIMARY KEY ("id");
DROP SEQUENCE "User_id_seq"

CREATE TABLE "Tweet" (
    "id" TEXT NOT NULL,
    "userId" TEXT NOT NULL,
    "content" TEXT NOT NULL,
    "tweetId" TEXT NOT NULL,

    PRIMARY KEY ("id")
)

CREATE TABLE "TweetMentions" (
    "id" TEXT NOT NULL,
    "tweetId" TEXT NOT NULL,
    "userId" TEXT NOT NULL,

    PRIMARY KEY ("id")
)

CREATE TABLE "TweetLikes" (
    "id" TEXT NOT NULL,
    "tweetId" TEXT NOT NULL,
    "userId" TEXT NOT NULL,

    PRIMARY KEY ("id")
)

ALTER TABLE "Tweet" ADD FOREIGN KEY("userId")REFERENCES "User"("id") ON DELETE CASCADE ON UPDATE CASCADE

ALTER TABLE "Tweet" ADD FOREIGN KEY("tweetId")REFERENCES "Tweet"("id") ON DELETE CASCADE ON UPDATE CASCADE

ALTER TABLE "TweetMentions" ADD FOREIGN KEY("tweetId")REFERENCES "Tweet"("id") ON DELETE CASCADE ON UPDATE CASCADE

ALTER TABLE "TweetMentions" ADD FOREIGN KEY("userId")REFERENCES "User"("id") ON DELETE CASCADE ON UPDATE CASCADE

ALTER TABLE "TweetLikes" ADD FOREIGN KEY("tweetId")REFERENCES "Tweet"("id") ON DELETE CASCADE ON UPDATE CASCADE

ALTER TABLE "TweetLikes" ADD FOREIGN KEY("userId")REFERENCES "User"("id") ON DELETE CASCADE ON UPDATE CASCADE

ALTER TABLE "User" ADD FOREIGN KEY("userId")REFERENCES "User"("id") ON DELETE SET NULL ON UPDATE CASCADE
```

## Changes

```diff
diff --git schema.prisma schema.prisma
migration 20201126164314-create-users..20201126202633-tweet-structure
--- datamodel.dml
+++ datamodel.dml
@@ -2,21 +2,51 @@
 // learn more about it in the docs: https://pris.ly/d/prisma-schema
 datasource db {
   provider = "postgresql"
-  url = "***"
+  url = "***"
 }
 generator client {
   provider = "prisma-client-js"
 }
 model User {
-  id    Int     @id @default(autoincrement())
-  email String  @unique
-  name  String?
+  id          String          @id @default(uuid())
+  email       String          @unique
+  name        String?
+  password    String
+  followers   User[]          @relation("UserFollowers")
+  Follower    User?           @relation("UserFollowers", fields: [userId], references: [id])
+  userId      String?
+  tweets      Tweet[]
+  mentionedIn TweetMentions[]
+  likedTweets TweetLikes[]
+}
-  following   User[] @relation("following")
-  User        User?  @relation("following", fields: [followingId], references: [id])
-  followingId Int?
+model Tweet {
+  id           String          @id @default(uuid())
+  from         User            @relation(fields: [userId], references: [id])
+  userId       String
+  content      String
+  responseFrom Tweet           @relation("TweetComments", fields: [tweetId], references: [id])
+  comments     Tweet[]         @relation("TweetComments")
+  tweetId      String
+  mentions     TweetMentions[]
+  likes        TweetLikes[]
+}
+model TweetMentions {
+  id      String @id @default(uuid())
+  tweet   Tweet  @relation(fields: [tweetId], references: [id])
+  user    User   @relation(fields: [userId], references: [id])
+  tweetId String
+  userId  String
 }
+
+model TweetLikes {
+  id      String @id @default(uuid())
+  tweet   Tweet  @relation(fields: [tweetId], references: [id])
+  user    User   @relation(fields: [userId], references: [id])
+  tweetId String
+  userId  String
+}
```

