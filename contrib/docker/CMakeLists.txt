
find_program(docker NAMES docker)
find_program(gpg NAMES gpg)

if (docker)

# Main build & runtime image 
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/ubuntu-18.04.dockerfile" ubuntu-18.04/dockerfile COPYONLY)
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/entrypoint.sh" ubuntu-18.04/entrypoint.sh COPYONLY)
add_custom_target(
  "docker-ubuntu-18.04"
  # COMMAND "${CMAKE_COMMAND}" --build . --target package_source
  COMMAND ${CMAKE_COMMAND} -E env DOCKER_BUILDKIT=1
    ${docker} build --target package --output type=local,dest=. ubuntu-18.04
  COMMAND ${CMAKE_COMMAND} -E env DOCKER_BUILDKIT=1
    ${docker} build ubuntu-18.04 --squash 
      --tag "frepple-ubuntu-18.04:${PROJECT_VERSION}"
      --tag "ghcr.io/frepple/frepple:${PROJECT_VERSION}"
      --tag "frepple-ubuntu-18.04:latest"
  )
add_custom_target(
  "docker-up"
  COMMAND ${CMAKE_COMMAND} -E env DOCKER_BUILDKIT=1
    ${docker} compose 
      --project-directory "${CMAKE_CURRENT_SOURCE_DIR}" 
      --project-name frepple 
      up    
      --detach
)
add_custom_target(
  "docker-down"
  COMMAND ${CMAKE_COMMAND} -E env DOCKER_BUILDKIT=1
    ${docker} compose
      --project-directory "${CMAKE_CURRENT_SOURCE_DIR}" 
      --project-name frepple
      down
)

# Build-only images
foreach(dist ubuntu-20.04 opensuse-15.1 centos-8 debian-10)
  configure_file(${CMAKE_CURRENT_SOURCE_DIR}/${dist}.dockerfile ${dist}/dockerfile COPYONLY)
  configure_file(${CMAKE_CURRENT_SOURCE_DIR}/entrypoint.sh ${dist}/entrypoint.sh COPYONLY)
  add_custom_target(
    "docker-${dist}"
    # COMMAND "${CMAKE_COMMAND}" --build . --target package_source
    COMMAND ${CMAKE_COMMAND} -E env DOCKER_BUILDKIT=1
      ${docker} build --target package --output type=local,dest=. ${dist}
    )
endforeach()

set(RHEL_USER  "Red Hat user name")
set(RHEL_PASSWORD "Red Hat password")
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/rhel-8.dockerfile rhel-8/dockerfile COPYONLY)
#if(gpg)
#$(GPG) --export-secret-keys -a devops@frepple.com >gpg_key
#endif()
add_custom_target(
  "docker-rhel-8"
  #COMMAND "${CMAKE_COMMAND}" --build . --target package_source
  COMMAND ${CMAKE_COMMAND} -E env DOCKER_BUILDKIT=1
    ${docker} build 
      --target package 
      --output type=local,dest=.
      --build-arg RHEL_USER=${RHEL_USER}
      --build-arg RHEL_PASSWORD=${RHEL_PASSWORD} 
      rhel-8
  )

endif()
