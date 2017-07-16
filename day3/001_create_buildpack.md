## Creating a detect script

1. Create a new folder for the buildpack:
  ```exec
  mkdir ~/custom_buildpack
  cd ~/custom_buildpack
  mkdir bin
  ```
Now you should refresh the file directory to see the created folder. 

2. Create the file `bin/detect` with the following content:
  ```file=~/custom_buildpack/bin/detect
  #!/usr/bin/env bash
  # bin/detect <build-dir>

  if [ -f $1/Staticfile ]; then
    echo "staticfile" && exit 0
  else
    echo "no" && exit 1
  fi
  ``` 
## Creating a compile script

1. Create the file `bin/compile` with the following content:
  ```file=~/custom_buildpack/bin/compile
  #!/usr/bin/env bash
  # bin/compile <build-dir> <cache-dir>

  set -e            # fail fast

  status() {
    echo "-----> $*"
  }

  # Configure directories
  build_dir=$1
  cache_dir=$2

  status "Moving application content into public folder"
  cd $build_dir
  mkdir public
  shopt -s extglob dotglob
  mv !(public) public 

  status "Installing static web server"
  wget -q -O static-fileserver  https://github.com/s-matyukevich/static-fileserver/blob/master/bin/static-fileserver?raw=true 
  chmod +x static-fileserver

  status "Creating boot script"
  cat > boot.sh << \EOF
  setsid $HOME/static-fileserver -p $PORT -d $HOME/public > out 2>out.err
  EOF
  chmod +x boot.sh

  echo "static"
  exit 0

  ```
## Creating a release script

1. Create the file `bin/release` with the following content:
  ```file=~/custom_buildpack/bin/release
  #!/usr/bin/env bash

  cat << EOF
  default_process_types:
    web: sh boot.sh 
  EOF
  ```
## Package and upload the buildpack

1. Install `zip`:
  ```exec
  sudo apt-get -y install zip
  ```

2. Add executable permissions:
  ```exec
  chmod +x ~/custom_buildpack/bin/*
  ```

3. Package the buildpack:
  ```exec
  cd ~
  zip -r custom_buildpack.zip custom_buildpack/
  ```

4. Create new space and target it.
  ```exec
  cf create-space -o cf-training test-space
  cf target -o cf-training -s test-space
  ```

5. Upload the buildpack:
  ```exec
  cf create-buildpack custom_buildpack custom_buildpack.zip 1
  ```

6. See the available buildpacks:
  ```exec
  cf buildpacks
  ```
## Verify the buildpack

1. Download a static application:
  ```exec
  cd ~
  git clone https://github.com/s-matyukevich/iot-dashboard
  ``` 

2. Add `Staticfile` to the appliction to make the buildpack recognize it:
  ```exec
  cd ~/iot-dashboard
  touch Staticfile
  ```

3. Deploy the application:
  ```exec
  cf push static
  ```
