---
{"dg-publish":true,"permalink":"/goodbye-docker-desktop-hello-orb-stack/","created":"2025-07-12T07:00:55.689-07:00","updated":"2025-07-12T07:11:45.387-07:00"}
---

My partner runs a small software company. In a recent update, they shared:

> We’ve dropped Docker Desktop for Mac in favor of OrbStack. We’ve been using containers for engineering work for a while and Docker has never felt good. OrbStack does everything Docker for Mac does and more, costs less, and is easier to use.

Funny enough at the same time, my Docker Desktop was having a _time_:
![Pasted image 20250712070420.png](/img/user/Pasted%20image%2020250712070420.png)
I'm really not sure how Docker Desktop lost a long running database container, but it did and now I have an excuse to try a different container orchestrator. It's job is to run Docker containers and it can't even do that.

After running the built-in OrbStack migration, I ran `docker ps` and it came back super fast. I didn't do a timer comparison but I've been noticing using `docker` commands getting slower. 

Well this is amazing. I am sold. It boots so fast, the UI is snappy and clean, and everything feels better. Immediately signed up for a Pro account. 

I'm glad people are still making great foundational software. Go team OrbStack!