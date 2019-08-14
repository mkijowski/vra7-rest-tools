#!/bin/bash
#
## vra7-rest-tools/vra7-rest.sh
## Author: matthewkijowski@gmail.com
## Full docs: https://github.com/mkijowski/vra7-rest-tools
## Shoutout: Ryan Kelly http://www.vmtocloud.com/how-to-delete-a-deployment-with-the-vra-7-rest-api/
##
## Please read comments and README.md, some functions are dangerous!
## 
## As is this file is meant to be sourced and provides simpler bash commands
## for interfacing with a vRealize appliance via the REST api.
# 
#variables
## VRA is the hostname of your vRA appliacne
VRA=vcsl.wright.edu
## TENANT is the tenant your are trying to access resources from
TENANT=wsu
## USERNAME is the name of a user with the required permissions to the 
## resources and actions you want to perform
USERNAME=mkijowski

main () {

## Get an auth token from VCSL
TOKEN=$(get_token)
}


## I can't warn you enough, this will destroy ALL deployments in the environment...
destroy_all () {
    for i in $(list_deployments); do
        destructor $i
        sleep 2
    done
}

## Debug function for arbitrary rest commands
## Expects one argument passed, which is the REST api link you are trying to test.
## Does NOT work with REST api calls which require post data
debug_rest () {
    curl -s --insecure \
        -H "Accept: application/json" \
        -H "Authorization: Bearer "$TOKEN"" \
        https://$VRA/$1 \
        | python -m json.tool
}

## List all resources
list_deployments () {
    for i in $(debug_rest catalog-service/api/consumer/resources/types/composition.resource.type.deployment/?limit=500 \
            | python -m json.tool \
            | grep  "^            \"id\": \"" \
            | sed 's/            \"id\": \"//' \
            | sed 's/",//')
    do
        echo $i
    done
}

## List available actions for specified resource $1
## Assuming there is only the destroy action available
## !!! warning, this assumpition may not always be correct !!!
list_actions () {
    curl -s --insecure \
        -H "Accept: application/json" \
        -H "Authorization: Bearer "$TOKEN"" \
        https://$VRA/catalog-service/api/consumer/resources/$1/actions \
        | python -m json.tool \
        | grep \"id\": \
        | sed 's/            "id": "//' \
        | sed 's/",//'
}

## Get token information securely
get_token () {
    curl -s --insecure \
        -H "Accept: application/json" \
        -H 'Content-Type: application/json' \
        --data "$(generate_post_data)" \
        https://$VRA/identity/api/tokens \
        | python -m json.tool \
        | grep id \
        | sed 's/    "id": "//' \
        | sed 's/",//'
}

## Generates post data when needed
generate_post_data () {
    read -s -p "Password: " PASSWORD
    cat <<EOF
{
"username":"$USERNAME",
"password":"$PASSWORD",
"tenant":"$TENANT"}" \
}
EOF
}

## Generate Post data for destroy action
generate_destroy_json () {
    cat <<EOF
{
    "type": "com.vmware.vcac.catalog.domain.request.CatalogResourceRequest",
    "resourceId": "$1",
    "actionId": "f9343b6b-de96-4126-87ac-84c4dc710fc0",
    "description": null,
    "data": {
        "description": null,
        "reasons": null
    }
}
EOF
}

## Get destroy json information.
## needed global variables: 
##      $VRA is link to vra appliance
##      $TOKEN is a valid vra token (only good for 24 hours)
## needed arguments:
##      $1 is the id to the deployment
##      $2 is the id to the destroy action for the above deployment
## !!!!! TODO: need to get the destroy action in the destructor
get_destroy () {
    curl -s --insecure \
        -H "Accept: application/json" \
        -H "Authorization: Bearer "$TOKEN"" \
        https://$VRA/catalog-service/api/consumer/resources/$1/actions/$2/requests/template \
        | python -m json.tool > /tmp/Destroy.json
}

## Destroys a deployment.
## needed global variables: 
##      $VRA is link to vra appliance
##      $TOKEN is a valid vra token (only good for 24 hours)
## needed arguments:
##      $1 is the id to the deployment
##      $2 is the id to the destroy action for the above deployment
## !!!!! TODO: need to get the destroy action in the destructor
destructor () {
    curl -s --insecure \
        -H "Accept: application/json" \
        -H "Authorization: Bearer "$TOKEN"" \
        https://$VRA/catalog-service/api/consumer/resources/$1/actions/f9343b6b-de96-4126-87ac-84c4dc710fc0/requests \
        --data "$(generate_destroy_json $1)" \
        --verbose \
        -H "Content-Type: application/json" \
        | python -m json.tool
}

main

