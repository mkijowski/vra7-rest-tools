### VRealize Automation 7 Rest Api tools

The purpose of this script was initially to be able to delete multiple
deployments automatically and is now doing so.  To do this several steps need to
be taken before issuing a delete call.  (please change the variables as
necessary at the top).

1. Get a token from the VRA appliance via `get_token`
2. Check if token works for making request of the REST api `list_deployments`
3. Find a way to send post data with variables in it, the unique formatting of
   post data screws with Bash variables so I create all needed post data
   programatically with `generate_post_data` and `generate_destroy_json`
4. Identify which resources you want to destroy, which I again used
   `list_deployments`
5. Identify the ID of the destroy action for that resource `list_actions`.
   Note: `list_actions` assumes the only available action is destroy.  This
   likely needs to be changed but works in my instance.
   Note 2: All Deployment resources in my environment share the same destroy
   action ID, this may not be the case in other environments AND certainly isnt
   the case for other resources like machines.  I have hard-coded this ID in, it
   will likely need changing elsewhere.
6. Get the destroy action JSON information with `get_destroy`
7. Check this info against the post data created by `generate_destroy_json`
8. If this is all good simply run the `destructor` and pass it the ID of the
   deployment you want destroyed.

### Resources
This was mostly templated from [Ryan Kelly's](http://www.vmtocloud.com/author/rkelly/)
work, specifically [How to delete a deployment with the vra 7 rest api](http://www.vmtocloud.com/how-to-delete-a-deployment-with-the-vra-7-rest-api/).

If you are trying to add to this you will probably need the VRA Rest API guide:
[https://code.vmware.com/apis/39/vrealize-automation](https://code.vmware.com/apis/39/vrealize-automation)
