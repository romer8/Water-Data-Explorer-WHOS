FROM tethysplatform/tethys-core:4.0.0

#########
# SETUP #
#########

# # Speed up APT installs
RUN echo "force-unsafe-io" > /etc/dpkg/dpkg.cfg.d/02apt-speedup \
 && echo "Acquire::http {No-Cache=True;};" > /etc/apt/apt.conf.d/no-cache \
 && echo "Acquire::Check-Valid-Until false;" > /etc/apt/apt.conf.d/no-check-valid

# # Install APT Package
RUN apt-get clean && apt-get update --allow-releaseinfo-change -qq && apt-get -yqq install apt-utils gcc libgdal-dev g++ libhdf5-dev > /dev/null

# # Quiet pip installs
RUN mkdir -p $HOME/.config/pip && echo "[global]\nquiet = True" > $HOME/.config/pip/pip.conf

# # Clone repository
RUN mkdir /tethys_apps && cd /tethys_apps && git clone https://github.com/Aquaveo/Water-Data-Explorer-WHOS.git

RUN mkdir -p ${TETHYS_PERSIST}/keys
# ###########
# # INSTALL #
# ###########

# install mamba much better than conda ##
RUN ${CONDA_HOME}/envs/tethys/bin/conda install -n base -c conda-forge mamba

# install tethys with mamba ##
RUN /bin/bash -c "cd /tethys_apps/Water-Data-Explorer-WHOS \
  ;${CONDA_HOME}/envs/tethys/bin/mamba install --yes --file requirements.txt -c conda-forge"
#; ${CONDA_HOME}/envs/tethys/bin/mamba install --yes pip -c conda-forge \

# RUN cd /tethys_apps/Water-Data-Explorer-WHOS && /bin/bash -c "${CONDA_HOME}/envs/tethys/bin/tethys install -N -w"

# For some reason the subsprocess command in "/usr/lib/tethys/tethys/tethys_cli/install_commands.py", line 693, in the install_command function, it is not 
# using the /opt/conda/envs/tethys/bin/pip but it wans to use /usr/bin/pip which does not extis that is why the error
# FileNotFoundError: [Errno 2] No such file or directory: 'pip'
RUN cp /opt/conda/envs/tethys/bin/pip /usr/bin
RUN cp /opt/conda/envs/tethys/bin/pip3 /usr/bin


RUN /bin/bash -c "cd /tethys_apps/Water-Data-Explorer-WHOS \
  ; ${CONDA_HOME}/envs/tethys/bin/tethys install -N -w -v"
  # ; tethys install -N -w"

# RUN /bin/bash -c ". ${CONDA_HOME}/envs/tethys/bin/activate tethys \
#   ;  pip install psycopg2==2.8.6"

RUN /bin/bash -c "${CONDA_HOME}/envs/tethys/bin/pip install psycopg2==2.8.6"

######################################################
# CHANGE THE PROXY TIME REPLACING THE NGINX TEMPLATE #
######################################################

# CHOWN #
#########
RUN export NGINX_USER=$(grep 'user .*;' /etc/nginx/nginx.conf | awk '{print $2}' | awk -F';' '{print $1}') \
  ; find ${TETHYSAPP_DIR} ! -user ${NGINX_USER} -print0 | xargs -0 -I{} chown ${NGINX_USER}: {} \
  ; find ${WORKSPACE_ROOT} ! -user ${NGINX_USER} -print0 | xargs -0 -I{} chown ${NGINX_USER}: {} \
  ; find ${STATIC_ROOT} ! -user ${NGINX_USER} -print0 | xargs -0 -I{} chown ${NGINX_USER}: {} \
  ; find ${TETHYS_PERSIST}/keys ! -user ${NGINX_USER} -print0 | xargs -0 -I{} chown ${NGINX_USER}: {} \
  ; find ${TETHYS_HOME}/tethys ! -user ${NGINX_USER} -print0 | xargs -0 -I{} chown ${NGINX_USER}: {}


#########################
# CONFIGURE ENVIRONMENT #
#########################
EXPOSE 8080

##########################
# ADD THE EXTRA MIDDLEWARE 
##########################

ADD custom_middleware.py /usr/lib/tethys/tethys/tethys_portal/



######################################################
# CHANGE THE PROXY TIME REPLACING THE NGINX TEMPLATE #
######################################################

ADD nginx /usr/lib/tethys/tethys/tethys_cli/gen_templates/

################
# COPY IN SALT #
################

## Be sure to be inside the docker_files folder ##
ADD /salt/ /srv/salt/

# #####################################
# # ADDITIONAL DATABASE CONFIGURATION #
# #####################################
ADD fix.sql $HOME
ADD configure_db.sh $HOME


# ######################
# # CONFIGURE FINAL_RUN #
# ######################
ADD final_run.sh $HOME

RUN echo ${TETHYS_PERSIST}
# #######
# # RUN #
# #######
CMD bash final_run.sh
