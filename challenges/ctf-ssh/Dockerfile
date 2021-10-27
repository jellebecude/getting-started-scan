# For more basic SSH challenges go to: https://jrtechs.net/open-source/teaching-ssh-through-a-ctf
FROM debian:bullseye-slim

RUN apt-get update && apt-get install -y --no-install-recommends \ 
    openssh-server \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN useradd -ms /bin/bash alice

COPY ctf.md /home/alice/ctf.md

RUN echo "alice:alice" | chpasswd

RUN service ssh start

EXPOSE 22

CMD ["/usr/sbin/sshd","-D"]