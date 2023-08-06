# Docker API - CMD

The Docker Registry API is not very extensive and not very useful if you want to collect data from it. 

 1. It has basic features for getting a list of tags. However this list has no additional information like creation/modified date as an example. The sorting of the list is also in alphabetic order rather than sorted by date. If you want the 10 newest tags then you are out of luck. 
 
 2. There are two manifest types. One is an actual manifest that points to a specific image while the other is a manifest list that points to several other manifests. The list contains useful information like the OS and Arch that each manifest is meant for. The actual manifest files however contains no such information. You will have to find those in the blob files. A date is also not available in iether of these. Again you will have to go further and base this on when the image was created. 
 
 3. You cannot in any way connect an image to tags pointing to it. You can find images from tags, but there is no reversing this. 
 
 
### docker-api

This script does what the API cannot, but at a cost due to the above mentioned issues. It can display information about images, connect tags and images and more. But in order to do so it needs to collect and connect all of the data associated with a project. Becuase everything is spread around in manifest lists, manifests, tags and blobs without any proper connections between them, this script will go through everything. Every single tag, every manifest that the tag points to and all blob files from the manifests. This is all cached locally and the script will then work off-line using the cached data. It can however refresh data much faster once it's done. 90% of this overhead could be avioded simply by having the Docker API include a date to each tag or at a minimum having it sort the tag list by date. Sadly this is not the case. 

__What the Docker API is missing__

 - The ability to get a single manifest from a tag like `latest`, get the ID of the 3 to 10 images from that manifest and then query the tag names accociated with each of those images. 
 - The API does have a pagination feature. If the tag list was listed based on creation/modified date, this feature could be used to limit the amount of tags to a few useful ones. Most people will have no need to get the last 3 years of tags and images. 
 - Alternatively to the above, have creation/modified date present for each tag in the tag list. This would allow anyone to sort by date manually. It would still require the full list to be downloaded, but avoids having to query every manifest and blob for that list.  
 
One of the last two suggestions would be the most useful, as the first one could be somewhat overcome by som date comparison magic. 


### Usage

```
Usage: docker-api [OPTIONS] <MODULE> <CMD> [<ARGS>,...]

Modules:
  registry                           Run commands against the docker registry
  library                            Manage cached library information
  inspect                            Inspect manifest and blob json data from registry

Registry Commands:
  refresh <repo> [<reference> ...]   Refresh the offline registry data
  info <repo> <reference>            Get info about a tag or image
  list <repo> <reference>            List tags or images

Library Commands:
  init <repo>                        Initialize a new library
  info <repo>                        Get info about a library
  list                               List all initialized libraries

Inspect Commands:
  manifest <repo> <reference>        Retrieve a manifest
  blob <repo> <reference>            Retrieve a blob

Registry Options:
  --os <string>                      Only show supported OS
  --arch <string>                    Only show supported architectures
  --ere <regexp>                     Only show/refresh tags matching this pattern

Library Options:
  --alias <string>                   Add an alias that can be used instead of the full repository name

 *  When running the 'registry list' command the keyword 'tags' and 'images' can be used instead of a reference. You can then further use the '--os' and '--arch' options to sort the output.

 *  When running the 'registry refresh' command the keyword 'all' can be used to refresh all tags. By default it will only look for new/missing tags during a refresh.

```


### Example

An example of a use case: Checking if a container needs to be updated.

```sh
# Only refresh the 'latest' tag
docker-api registry refresh portainer/portainer-ce latest

# If the current image is not part of the 'latest' tag, then this tag has been updated
if ! docker-api registry list portainer/portainer-ce latest | grep -q $(docker inspect portainer -f '{{.Image}}'); then
    echo "New update is available"
fi
```

