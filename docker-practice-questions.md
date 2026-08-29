# Docker Practice Questions

This is a practice sheet, not an answer sheet.

Every question below has a short pointer telling you what to think about and what
to search for. Do the searching yourself, write your own answer, and test it on a
real machine wherever you can. The answer sheet will be shared afterwards.

Rules for getting value out of this:

1. Write your answer before you search.
2. Then search, and correct what you got wrong.
3. Then prove it with a command on a real Docker host.
4. Mark any question where your first answer was wrong. Those are the ones to revise.

---

## Section 1. Fundamentals

**1. What is the difference between an image and a container?**
Direction: build artifact versus running process. Search for image layers and the
container writable layer.

**2. Can two containers run from the same image at the same time? What is different between them?**
Direction: think about what is shared read only and what is per container.

**3. What exactly happens when you run docker run?**
Direction: trace the path from the CLI to the daemon to containerd to runc to the
kernel. Search for Docker architecture components.

**4. Is a container a lightweight virtual machine? Defend your answer.**
Direction: think about kernels. How many kernels are running in each case.

**5. Which Linux kernel features make containers possible?**
Direction: namespaces and cgroups. Learn which namespace isolates what.

**6. Name the main Linux namespaces and say what each one isolates.**
Direction: pid, mnt, net, uts, ipc, user. One line each.

**7. What do cgroups control that namespaces do not?**
Direction: visibility versus consumption. Think walls versus budget.

**8. If a process inside a container is PID 1, what PID does it have on the host?**
Direction: a different one. Search for pid namespace mapping. Try it on a Linux host
with ps on both sides.

**9. What is the difference between the Docker CLI and the Docker daemon?**
Direction: client and server. Search for the Docker socket and the REST API.

**10. What is containerd and what is runc? Why are there two?**
Direction: lifecycle management versus actually creating the container. Search for OCI runtime spec.

**11. Kubernetes removed Docker Engine as a runtime. Do images built with Docker still work?**
Direction: think about standards. Search for OCI image specification and dockershim removal.

**12. What is the difference between docker stop and docker kill?**
Direction: which signal is sent, and whether the process gets a chance to shut down.

**13. What happens to a container when the process it runs exits?**
Direction: the container lifecycle is tied to PID 1 inside it.

**14. Why does a container running only a shell script exit immediately sometimes?**
Direction: foreground versus background processes. Think about what PID 1 is doing.

**15. What is the difference between docker create, docker start and docker run?**
Direction: one of them is the other two combined.

**16. What does docker exec do that docker attach does not?**
Direction: new process versus connecting to the existing one.

**17. What information does docker inspect give you that docker ps does not?**
Direction: try it. Look at the full JSON for a running container.

**18. What is the difference between docker pause and docker stop?**
Direction: search for the freezer cgroup.

---

## Section 2. Images and layers

**19. What creates a new layer in an image?**
Direction: which Dockerfile instructions change the filesystem and which only change metadata.

**20. Why are layers shared between images and what does that save?**
Direction: content addressing. Search for image layer digests and storage drivers.

**21. What is a union filesystem and why does Docker need one?**
Direction: search for overlay2, lower dir, upper dir, merged dir.

**22. If you delete a file in layer 8 that was added in layer 4, is it gone from the image?**
Direction: search for whiteout files. Then test it by extracting the image with docker save.

**23. Why does deleting files in a later RUN not reduce image size?**
Direction: same as above. This one matters for security, not just size.

**24. How would you prove that a secret deleted in a later layer is still recoverable?**
Direction: docker save, then extract the tar and search the layer contents.

**25. What is the build cache and when does it get invalidated?**
Direction: the instruction, its inputs, and everything below the first miss.

**26. Why should COPY requirements.txt come before COPY of application code?**
Direction: think about what changes daily versus monthly. Then time both orderings.

**27. What is the build context and what gets sent to the daemon?**
Direction: the directory you pass to docker build. Search for build context size warnings.

**28. What does .dockerignore do and why is it a security control as well as a speed control?**
Direction: think about .git, .env, credentials, and what COPY dot dot picks up.

**29. What is the difference between a tag and a digest?**
Direction: a label that can move versus bytes that cannot. Search for image immutability.

**30. Why is the latest tag considered dangerous?**
Direction: it is a name, not a promise. Think about what happens on a node restart weeks later.

**31. What does docker history show you and what can you learn from it?**
Direction: layer sizes and the instruction that created each one. Also environment variables.

**32. How do you find out which layer is making your image large?**
Direction: docker history sorted by size. Also search for dive as a tool.

**33. What is a dangling image and how do they accumulate?**
Direction: untagged images left after rebuilds. Search for docker image prune.

**34. What does docker system df tell you and when would you use it?**
Direction: reclaimable space across images, containers, volumes, build cache.

---

## Section 3. Dockerfile authoring

**35. Explain FROM, WORKDIR, COPY, RUN, ENV, EXPOSE, CMD and ENTRYPOINT in one line each.**
Direction: write it out yourself before searching. Focus on build time versus run time.

**36. What is the difference between CMD and ENTRYPOINT?**
Direction: what happens when you append arguments to docker run.

**37. When would you use ENTRYPOINT with CMD together?**
Direction: think about a container that behaves like a single executable with default arguments.

**38. What is the difference between the exec form and the shell form of CMD?**
Direction: JSON array versus a plain string. Search for what wraps your process in sh minus c.

**39. Why does the shell form of CMD cause slow container shutdown?**
Direction: signal handling. Which process gets SIGTERM and which one ignores it.

**40. Does EXPOSE publish a port?**
Direction: no. Find out what it actually does and what publishes a port.

**41. What is the difference between ARG and ENV?**
Direction: build time versus run time, and which one is visible in the final image.

**42. Why is putting a secret in an ARG still unsafe?**
Direction: search for build arguments in image history and metadata.

**43. What is the difference between COPY and ADD? Which should you use?**
Direction: ADD does more, and that is the problem. Search for ADD remote URL and auto extraction.

**44. What does COPY with the chown flag do and why does it matter with a non root user?**
Direction: file ownership inside the image versus the user the process runs as.

**45. Why does RUN cd /app not persist to the next instruction?**
Direction: each RUN is a new layer and a new shell. Use WORKDIR instead.

**46. Why combine apt-get update and apt-get install in one RUN?**
Direction: cache invalidation and stale package indexes. Search for the apt cache busting problem.

**47. What is a HEALTHCHECK in a Dockerfile and what does Docker do when it fails?**
Direction: the honest answer to the second half surprises people. Test it.

**48. Why might a HEALTHCHECK using curl fail in a hardened image?**
Direction: what is actually installed in a slim or distroless base.

**49. What does the USER instruction do and where in the Dockerfile should it go?**
Direction: near the end. Think about what breaks if you put it too early.

**50. Why should USER specify a numeric UID rather than a username?**
Direction: think about what an orchestrator can verify before starting the container.

**51. What is a .dockerignore mistake that leaks credentials?**
Direction: missing .env or .git, combined with COPY dot dot.

**52. What does PYTHONDONTWRITEBYTECODE do and why does it matter for a read only filesystem?**
Direction: where Python tries to write pyc files.

**53. Why should application logs go to stdout rather than a file inside the container?**
Direction: think about where the log collector looks, and what happens when the container is removed.

---

## Section 4. Multi-stage builds and image size

**54. What problem does a multi-stage build solve?**
Direction: you need build tools to build and you do not want to ship them.

**55. How does COPY with the from flag work?**
Direction: copying from a named earlier stage. Search for named build stages.

**56. Why is a multi-stage build better than removing packages at the end?**
Direction: layers are immutable. Go back to question 23.

**57. Compare the sizes of python full, python slim, python alpine and distroless.**
Direction: pull them and run docker images. Then explain the trade-off of each.

**58. What is the risk of using alpine for a Python application?**
Direction: musl libc versus glibc. Search for manylinux wheels and alpine build times.

**59. What is a distroless image and what do you lose by using one?**
Direction: no shell, no package manager. Think about how you would debug it.

**60. When is FROM scratch appropriate?**
Direction: statically linked binaries. Why does that rule out most Python and Java.

**61. How do you debug a distroless container that has no shell?**
Direction: search for ephemeral debug containers and kubectl debug.

**62. Can you build only one stage of a multi-stage Dockerfile? How?**
Direction: search for the target flag on docker build.

**63. Your image is 1.2 GB. List five things you would check first.**
Direction: base image, build tools left in, package cache, copied context, missing dockerignore.

**64. How do you build an image for a different CPU architecture than your laptop?**
Direction: search for the platform flag and buildx. This matters on Apple Silicon.

**65. What happens if you push an arm64 image and run it on an amd64 node?**
Direction: search for exec format error.

---

## Section 5. Networking

**66. What are the default Docker network drivers and when would you use each?**
Direction: bridge, host, none, overlay, macvlan.

**67. What is the difference between the default bridge network and a user defined bridge network?**
Direction: one of them gives you DNS resolution by container name. Test both.

**68. How does one container reach another by name?**
Direction: search for the Docker embedded DNS server and its address.

**69. What is the address of Docker's internal DNS resolver inside a container?**
Direction: look at /etc/resolv.conf inside a container on a user defined network.

**70. What does the p flag do and what do the two numbers mean?**
Direction: host port and container port. Which one is arbitrary.

**71. Can two containers both listen on port 8080? Explain.**
Direction: network namespaces. Then think about the host side.

**72. What is a network alias and what does DNS return when several containers share one?**
Direction: search for network-alias and DNS round robin.

**73. What are the limits of Docker DNS round robin as a load balancing mechanism?**
Direction: does it health check. Does it remove broken containers. Does the client cache.

**74. What does host network mode do and what do you give up?**
Direction: no network namespace isolation. Think about port conflicts and security.

**75. How do you inspect which containers are attached to a network?**
Direction: docker network inspect.

**76. Why can a container reach the internet but nothing can reach it from outside without p?**
Direction: search for Docker iptables rules and NAT.

**77. How does a container reach a service running on the host machine?**
Direction: search for host.docker.internal and its platform limitations.

---

## Section 6. Storage and data

**78. What happens to data written inside a container when the container is removed?**
Direction: the writable layer. Test it.

**79. What is the difference between a volume and a bind mount?**
Direction: who manages the storage and where it lives.

**80. When would you use a tmpfs mount?**
Direction: sensitive scratch data, and read only root filesystems.

**81. Why are containers described as ephemeral, and why is that a feature not a limitation?**
Direction: think about what disposability enables in an orchestrator.

**82. Where should configuration live if not inside the image?**
Direction: environment variables, mounted files, secrets managers.

**83. If containers are ephemeral, how do you run a database in one?**
Direction: volumes for the data path. Then think about whether you should.

**84. What is the risk of bind mounting the Docker socket into a container?**
Direction: search for docker socket mount privilege escalation. This one is important.

---

## Section 7. Security and hardening

**85. Why is running a container as root a problem if it is not host root?**
Direction: write access inside the container, and what it enables in an escape chain.

**86. List everything an attacker gets if they achieve code execution in a full Debian based image.**
Direction: shell, package manager, compiler, network tools, writable filesystem.

**87. How does a non root user, a read only filesystem and no package manager change that picture?**
Direction: think about what an exploit chain needs at each step.

**88. What does the read only flag do and what usually breaks when you set it?**
Direction: anything that writes to disk. Search for combining it with tmpfs.

**89. What are Linux capabilities and why drop all of them?**
Direction: search for capabilities list, cap-drop, cap-add.

**90. What does no-new-privileges prevent?**
Direction: search for setuid binaries and privilege escalation.

**91. What does the privileged flag do and why is it almost always wrong?**
Direction: search for what privileged actually grants. It is more than people expect.

**92. Why set memory and CPU limits on a container even in a small environment?**
Direction: noisy neighbour, and what happens when one container consumes everything.

**93. What is the difference between an OS package vulnerability and an application dependency vulnerability?**
Direction: where each comes from, and how each is fixed.

**94. What does Trivy actually read inside an image to find vulnerabilities?**
Direction: package manifests. Search for how scanners build an inventory.

**95. Why does moving from a full base to a slim base drop the vulnerability count without patching anything?**
Direction: the vulnerable package is absent rather than fixed.

**96. What is an SBOM and why are organisations asking for one?**
Direction: software bill of materials. Think about the day a new CVE is published.

**97. What severity levels would you fail a build on, and why not fail on all of them?**
Direction: think about what happens to a scanner that developers learn to ignore.

**98. Name three ways a credential can end up inside an image.**
Direction: ENV, COPY of a config file, ARG used in a RUN.

**99. A credential was committed into an image and later deleted. What is the correct remediation?**
Direction: it is not deleting the file. Think about rotation.

**100. Why is image signing useful and what does it protect against?**
Direction: search for cosign, sigstore, supply chain attacks.

**101. What is tag immutability in a registry and what attack does it prevent?**
Direction: someone pushing different bytes to the same tag.

**102. Why is scanning at build time not enough on its own?**
Direction: new vulnerabilities are published against images you already shipped.

**103. What is the shared kernel risk with containers, and what is the mitigation if you cannot accept it?**
Direction: search for kernel escape, gVisor, Firecracker, Kata containers.

---

## Section 8. Registries and distribution

**104. What is a registry and why can you not just copy images between machines?**
Direction: you can, with docker save and load. Then explain why nobody does.

**105. What is the difference between docker save and docker push?**
Direction: a tar file versus a registry protocol.

**106. What happens when you push an image whose base layers are already in the registry?**
Direction: watch the output of a push carefully.

**107. What are the differences between Docker Hub and a private registry like ECR?**
Direction: access control, rate limits, scanning, cost, network path.

**108. What is the Docker Hub anonymous pull rate limit and when does it bite you?**
Direction: think about many nodes behind one NAT gateway pulling at once.

**109. How does authentication to ECR work and why is there no stored password?**
Direction: search for get-login-password and short lived tokens.

**110. What is an image lifecycle policy and what problem does it solve?**
Direction: untagged layers accumulating and billing forever.

**111. Why should a production deployment reference a digest rather than a tag?**
Direction: unambiguous by construction. Think about caching on nodes.

---

## Section 9. Troubleshooting

**112. A container exits immediately after docker run. How do you find out why?**
Direction: docker ps -a for the exit code, docker logs for the output.

**113. docker logs shows nothing. What are the likely causes?**
Direction: the app logs to a file, or output is buffered. Search for PYTHONUNBUFFERED.

**114. You get permission denied when running docker as a normal user. What is wrong?**
Direction: group membership and the Docker socket. Also why re-login is needed.

**115. A container is running but you cannot reach it on the port you expected.**
Direction: check the p mapping, check what address the app binds to, check the firewall.

**116. An application works with docker run but fails inside a container with a connection refused to itself.**
Direction: binding to 127.0.0.1 versus 0.0.0.0.

**117. Two containers on the same host cannot resolve each other by name.**
Direction: default bridge versus user defined network.

**118. The build fails at pip install with a network timeout. What do you check?**
Direction: daemon network, proxy settings, DNS inside the build.

**119. The build is very slow and shows a huge context transfer at the start.**
Direction: .dockerignore, and what is in the build directory.

**120. A container starts as root fine but crashes after you add USER.**
Direction: file ownership. Search for chown in COPY.

**121. The container crashes when you add the read only flag.**
Direction: something is writing to disk. Find what and give it a tmpfs.

**122. docker exec fails with container not running.**
Direction: it exited. Check docker ps -a and logs with the previous flag where applicable.

**123. The image runs on your laptop but crashes on the server with exec format error.**
Direction: CPU architecture mismatch.

**124. Your disk is full on a Docker host. What do you clean and in what order?**
Direction: docker system df first, then prune with care. Know what prune deletes.

**125. A container is being killed with exit code 137. What does that mean?**
Direction: search for SIGKILL and OOM killer. Then check the memory limit.

**126. A container is running but the application inside is hung and returns errors.**
Direction: this is the interesting one. What does Docker do about it. Test with a HEALTHCHECK.

**127. How do you see resource usage of running containers live?**
Direction: docker stats.

**128. How do you find out what changed in a container's filesystem since it started?**
Direction: docker diff.

---

## Section 10. Docker Compose

**129. What problem does Compose solve that plain docker run does not?**
Direction: multiple services, ordering, networks, one command.

**130. What is the difference between an imperative command and a declarative file?**
Direction: telling a machine what to do versus recording what should be true.

**131. What does depends_on actually guarantee and what does it not?**
Direction: start order, not readiness. Search for depends_on with condition.

**132. What does docker compose up do when you change an image tag?**
Direction: recreate. Then ask whether there is a gap in service.

**133. Can Compose do a rolling update with no downtime?**
Direction: think about what it would need to own to do that.

**134. Can you scale a service in Compose if it publishes a fixed host port?**
Direction: try it. Explain the error.

**135. What is a restart policy and which policies exist?**
Direction: no, on-failure, always, unless-stopped. Know the difference between the last two.

**136. Does restart always fire when a HEALTHCHECK fails?**
Direction: test this. The answer surprises people and it matters.

---

## Section 11. Limits and boundaries

**137. Docker restarts a crashed container. What does it not restart?**
Direction: think about the machine.

**138. Your Docker host dies at 3 AM. What brings the application back?**
Direction: nothing. Explain why and what would be needed.

**139. You have ten Docker hosts and forty containers. Who decides which container runs where?**
Direction: you do, by hand. Explain why that does not scale.

**140. A container on host A needs to reach a container on host B by name. What happens?**
Direction: Docker DNS is per host. Explain the consequence.

**141. Docker knows a container is unhealthy. List everything it does about it.**
Direction: be precise and honest. This is a short list.

**142. What is a reconciliation loop and why does Docker not have one?**
Direction: desired state versus actual state, compared continuously.

**143. List four things Docker does well and four things it cannot do at all.**
Direction: this is the summary question. Packaging, isolation, speed, portability on one side.

**144. Why did Kubernetes win over Docker Swarm despite Swarm being simpler?**
Direction: governance, vendor neutrality, ecosystem. Think about why cloud providers adopted it.

---

## Section 12. Scenario based

**145. A developer says the application works on their laptop but fails in production. Both use the same image. What do you investigate?**
Direction: configuration, environment variables, secrets, network reachability, architecture, mounted files.

**146. A team's CI build takes eleven minutes and most of it is pip install. The code changes many times a day but dependencies rarely change. What do you fix?**
Direction: Dockerfile instruction ordering and layer caching.

**147. A security audit finds 180 high severity CVEs in your image. The application has two dependencies. Explain and fix.**
Direction: where the CVEs actually are, and how base image choice changes the number.

**148. An engineer suggests deleting the compiler in a final RUN to shrink the image. What do you tell them?**
Direction: layers are immutable. Explain what does work.

**149. A production container is compromised. What difference does it make whether it was running as root or as uid 10001 on a read only filesystem?**
Direction: walk the exploit chain step by step and say where each step fails.

**150. You need to run twelve small services on one machine. They need different Python versions. How do containers help and what would you have had to do without them?**
Direction: dependency isolation, and the alternative of virtualenvs or separate VMs.

**151. Your team wants to store the database password in the Dockerfile so deployments are simpler. Give three reasons not to, and one alternative.**
Direction: image inspect, layer extraction, image sharing. Alternative is runtime injection.

**152. An image tagged 1.4.0 behaves differently on two nodes. Both pulled the same tag. How is that possible and how do you prevent it?**
Direction: tag mutability, node image cache, imagePullPolicy. Prevention is immutable tags or digests.

**153. A container in production is using 8 GB of RAM and starving other services. What did you fail to configure, and what would you add?**
Direction: memory limits and cgroups.

**154. Your deployment process is docker compose up on a single EC2 instance. Write down the five failure modes you are exposed to.**
Direction: host failure, no rolling update, no rollback, no cross host scaling, no health based routing.

**155. Design the container build strategy for a Java service that needs Maven to build and a JRE to run. What does each stage contain?**
Direction: multi stage. Maven and JDK in stage one, JRE only plus the jar in stage two.

**156. A colleague mounts the Docker socket into a CI container so it can build images. What is the risk and what alternatives exist?**
Direction: socket access is effectively root on the host. Search for rootless builders, kaniko, buildkit.

**157. Your image passes the vulnerability scan on Monday and fails on Friday with no code change. Explain how.**
Direction: the vulnerability database changed, not your image.

**158. You are asked to reduce container startup time for a service that scales up during traffic spikes. What are your levers?**
Direction: image size, layer caching on nodes, base image, application startup time, readiness gating.

**159. A microservice needs to call another microservice. Compare how you would configure that address on a single Docker host, and what breaks when you move to many hosts.**
Direction: container name and embedded DNS, versus the need for cluster wide discovery.

**160. Write down the ten Dockerfile rules you would enforce in a code review checklist.**
Direction: this is your summary. Base image, pinning, ordering, dockerignore, non root, no secrets,
multi stage, exec form, healthcheck, logs to stdout.

---

## How to practise

Take one section per day. For each question, write your answer, then verify it with a
command. The questions marked as scenario based are the ones that come up in interviews,
and they are only answerable if you understand the sections before them.

Bring your wrong answers to the next session. Those are worth more than the right ones.
