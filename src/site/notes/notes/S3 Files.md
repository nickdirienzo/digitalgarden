---
{"dg-publish":true,"permalink":"/notes/s3-files/","created":"2026-04-07T14:51:34.513-07:00","updated":"2026-04-07T15:12:41.714-07:00"}
---

With AI agents depending on filesystems for searching, editing, and storing context, it was only a matter of time before AWS had a cloud-native answer.

Today is that day: AWS launched S3 Files ([All Things Distributed](https://www.allthingsdistributed.com/2026/04/s3-files-and-the-changing-face-of-s3.html), [AWS](https://aws.amazon.com/s3/features/files/)). 

This is a big deal. Agents work on filesystems. They `ls`, `grep`, `cat`. We've watched Claude Code and Codex shift the industry from semantic search back to grep, which spawned a wave of tools built as CLIs instead of MCP servers. As an industry we've learned that agents are most capable when they operate like developers do, on files.

S3 Files maps buckets and prefixes to NFS mounts. That means you can create filesystem sandboxes backed by S3 durability and attach them to Lambda, ECS, whatever. Lambda becomes stateless execution with a durable filesystem. 

Personalized agent contexts, scratchpads, durable working directories all on top of AWS primitives. That's an amazing foundation for anyone doing agentic work on AWS.

The cloud is catching up to how agents actually work.
