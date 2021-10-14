# 1. Getting started - Container Challenges

Getting started with container-based challenges on the Joint Cyber Range.

- [1. Getting started - Container Challenges](#1-getting-started---container-challenges)
  - [1.1. Challenges](#11-challenges)
    - [1.1.1. Solve some challenges](#111-solve-some-challenges)
    - [1.1.2. Make your own CTF challenge](#112-make-your-own-ctf-challenge)
      - [1.1.2.1. Instructions container-based challenge](#1121-instructions-container-based-challenge)
    - [1.1.3. Add your challenges to CTFd](#113-add-your-challenges-to-ctfd)

- [Back to home](../README.md)

## 1.1. Challenges

CTF challenges basically run down to; IT/CYber Security puzzles, a digital scavenger hunt or escape room. Various types of CTF events exist, while CTFd makes it possible to host Jeopardy Style events.
Events focussed on professionals or internal training, try to simulate realistic Cyber Security scenarios. Attack/Defense games or Red vs Blue scenarios, is where the Joint Cyber Range will be heading to in the future.

### 1.1.1. Solve some challenges

In the zip file are some basic challenges included, from various CTF categories. Solve these to get a glimpse of how a CTF works.

Username: user
Password: user

**Note:** I wasn't able to deploy a container based challenge on CTFd. You can pull the image```dockerburthet/ctf-ssh``` and try to solve the challenge or build it yourself. This challenge Dockerfile could also be used as a template for future challenges you'll build, that need an SSH connection to the container.
**Title:** SSH
**Description:** Alice makes use of bad password hygiene on her server. Retrieve the content of the hidden message.


### 1.1.2. Make your own CTF challenge

Try to think of a basic CTF challenge that somebody else must be able to solve.

- Design and make a static text or upload based challenge first and save it in CTFd.
- Move on to creating a container-based challenge.

#### 1.1.2.1. Instructions container-based challenge

**1.** Create a dockerfile for the container(s) you want to include in the challenge. **Note:** All necessary files that it interacts with, must be included in the image.

**2.** Build and try your image. When you're satisfied, upload the container image to a container registry.

**3.** Package it all up as a docker-compose file and test. If it works, you can encode your docker-compose file

**4.** Upload the challenge and test if the deployment and flag input work. Export your static and container-based challenges and share it with somebody to solve.

### 1.1.3. Add your challenges to CTFd

You can login as admin and add your newly made challenges to the catalogue. Export your CTF event and share it, to get your challenges peer reviewed.
