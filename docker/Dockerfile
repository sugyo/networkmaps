FROM node

RUN apt-get update -y && apt-get upgrade -y && \
	apt-get install -y --no-install-recommends git jq && \
	apt-get -y clean && \
	ln -s /usr/local/bin/node /usr/bin/node

RUN mkdir -p \
	/var/lib/networkmaps/etc \
	/var/lib/networkmaps/users \
	/var/lib/networkmaps/diagrams \
	/var/lib/networkmaps/shapes \
	/var/lib/networkmaps/sendmail/queue \
	/var/lib/networkmaps/sendmail/sent
RUN ln -s /var/lib/networkmaps/etc /etc/networkmaps

WORKDIR /work 
RUN git clone https://github.com/pablomarle/networkmaps.git

WORKDIR /work/networkmaps
ADD patches /work/networkmaps/patches
RUN cat patches/* | patch -p1

RUN echo '{"socket":{"address":"127.0.0.1","port":3000},"users":{"admin_username":"admin","admin_password":"admin","register_self":false}}' | jq > /var/lib/networkmaps/etc/config.json && \
    ./server.js & echo $! > /tmp/server.pid && sleep 3 && \
    ./nm_admin_cli.js --server 127.0.0.1 --port 3000 login admin admin | awk '/Session ID:/{ print $3}' > /tmp/server.session && \
    ./nm_admin_cli.js --server 127.0.0.1 --port 3000 --session `cat /tmp/server.session` add_user user1@example.com User1 Example user1 && \
    ./nm_admin_cli.js --server 127.0.0.1 --port 3000 --session `cat /tmp/server.session` logout && \
    kill `cat /tmp/server.pid` && sleep 1 && \
    rm /tmp/server.session /tmp/server.pid && \
    jq .user < /var/lib/networkmaps/users/user_db.json

EXPOSE 3000

CMD ["./server.js"]
