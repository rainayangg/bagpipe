Bagpipe
================

### Getting Started with Bagpipe

Bagpipe is a tool which enables an ISP to express its BGP policy in a
domain-specific specification language and verify that its router configurations
implement this policy.

This README provides a quick start guide for Bagpipe, a more in depth discussion
can be found [here](http://konne.me/bagpipe).

To build up Bagpipe, first you should pull Docker image which contains the older version of Bagpipe using

    docker pull konne/bagpipe

To run this image:
    
    docker run --name new_bagpipe --entrypoint /bin/bash -ti konne/bagpipe
Then you will get into a container which has the older version of Bagpipe but it might not be what you want. 
If it is the case, you should compile this newer version of Bagpipe in the docker container. To do that:

First archive the older version:

    mv /bagpipe /old_bagpipe
    git clone https://github.com/uwplse/bagpipe
    cd /bagpipe

There are 3 things you need to take care of in order to compile the new Bagpipe. First, it relies on Space-Search which 
is not installed in the container by default. In order to satisfy the dependency, you need to install Space-Search using OPAM:

    wget https://raw.github.com/ocaml/opam/master/shell/opam_installer.sh -O - | sh -s /usr/local/bin
    sudo apt install unzip
    
    opam init -n --comp=4.01.0
    opam install coq.8.5.2
    opam update && opam install space-search.0.9.1
    
    export PATH=/root/.opam/4.01.0/bin:$PATH

Second, you need to reinstall racket because the current version in the container does not satisfy the dependency of the newer Bagpipe:

    wget http://mirror.racket-lang.org/installers/6.6/racket-6.6-x86_64-linux.sh -O install.sh && \
    chmod +x install.sh && \
    ./install.sh --in-place --create-links /usr --dest /usr/racket && \
    rm install.sh

Last, you need to reinstall rosette:

    cd /
    cd rosette && \
    sed -i "s/;(fprintf/(fprintf/g" rosette/solver/smt/smtlib2.rkt && \
    raco pkg remove rosette && \
    raco pkg install

After doing the above, you should be able to compile the newer bagpipe by running:

    make -C /bagpipe

Last but not least, if you want to newer Bagpipe to be the default choice, you should add the path:
    
    export PATH=/bagpipe/src/bagpipe/python/:$PATH


### Developing Bagpipe

You can also use Docker to start a Bagpipe development environment that has
all the right dependencies setup: 

    docker rm -f bagpipe; docker run --name bagpipe --entrypoint /bin/bash -v $(pwd):/bagpipe -ti bagpipe

Running the above command starts a shell in the development environment. You can
build the Bagpipe project with:

    make -C /bagpipe

From another terminal, you can connect to the development environment with your
local emacs installation:

    emacs /docker:bagpipe:/bagpipe/src/bagpipe/coq/Main/BGPSpec.v 

If your docker instance runs on another machine, you can connect to it with:

    emacs "/ssh:user@machine|docker:bagpipe:/bagpipe/src/bagpipe/coq/Main/BGPSpec.v"

Make sure your emacs has `ProofGeneral` and `docker-tramp` installed, and 
`enable-remote-dir-locals` must be set.

### Publishing Bagpipe

To push bagpipe to DockerHub run:

    docker login
    docker push konne/bagpipe-private

The name of your local image must match the name that you wish to push.
You can get the image back with

    docker pull

