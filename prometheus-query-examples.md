```
(clamp_min(
    sum(
        upload_to_object_container_duration_seconds_sum{cluster=~"$cluster"}
    ) by (object_storage_container_name)
    - 
    sum(
        upload_to_object_container_duration_seconds_sum{cluster=~"$cluster"} offset $__interval 
    ) by (object_storage_container_name)
, 0)) / (clamp_min(
    sum(
        upload_to_object_container_duration_seconds_sum{cluster=~"$cluster"}
    ) by (object_storage_container_name)
    - 
    sum(
        upload_to_object_container_duration_seconds_sum{cluster=~"$cluster"} offset $__interval 
    ) by (object_storage_container_name)
, 1))
```
