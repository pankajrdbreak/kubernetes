# Multi-container Pods

As name suggest a pod with more than one conatiner in it is called as Multicontainer pod.
The multi-container pods are the pods that contain two or more related containers that share resources like network space, shared volumes, etc and work together as a single unit. Basically, these helper processes or containers enhance the main containers by providing additional functionality.

In multicontainer pods there is content-generator which will create content continously and put it into shared volume and then this volume is accessed by main-container which will display the content.

The container is not get terminated after pulling the content.It will work as a side car for the main container
