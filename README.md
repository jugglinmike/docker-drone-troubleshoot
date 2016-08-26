# Troubleshooting Drone.io/Docker Compose integration

I'm trying to use my project's `docker-compose.yml` file in a Drone.io build.
This repository is a reduced case of the problem I am facing in my project.

Taking a cue from Drone's documentation, I have configured Drone to mount the
hosts's Docker socket. Now, when Drone runs a container from the
`docker-compose` image, it is able to create additional containers dynamically.

However, the containers inter-operate through Docker volumes, and these volumes
cannot be properly shared in this context.

In this project, running `docker-compose` "natively" produces the expected
output: the `db` directory is successfully mounted in the "web" container:

    $ docker-compose run --rm web
    /db:
    example.txt

    /home:

The same command, when issued by Drone configured to share the Docker socket,
fails to properly mount the directory:

    $ drone exec --trusted

    [DRONE] starting job #1
    $ docker-compose run --rm web
    /db:

    /home:
    [DRONE] finished job #1
    [DRONE] exit code 0
    [DRONE] job #1 passed
    [DRONE] build passed

"Traditional" use case (invoking `docker-compose` directly:

    .------- host ------.
    | $ docker-compose  |
    |                   |
    |         .- web -. |
    | ./db -> |  /db  | |
    |         '-------' |
    '-------------------'

"Advanced" usage (invoking `docker-compose` within Drone's container:

    .------- host -----------------------.
    | $ drone exec                       |
    |                                    |
    | .----- drone ------.               |
    | | $ docker-compose |               |
    | |                  |     .- web -. |
    | |      ./db -----> | ??? |  /db  | |
    | '------------------'     '-------' |
    '------------------------------------'

In this case, the directory to be shared is actually hosted within the Drone
container. So when the Docker Engine on the host (following instructions from
the Drone container) is told to mount the volume at `./db`, it is unable to
find the directory in question.

I'm not sure if this is exactly a bug in Drone, but I'm wondering if there is
some way to configure the systems in order to facilitate this sharing.

Thanks!
