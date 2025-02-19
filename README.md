# cyber_record
Cyber record file offline parse tool. You can use `cyber_record` to read messages from record file, or write messages to the record file.

## Quick start
First install "cyber_record" by the following command.
```
pip install cyber_record
// or update version
pip install cyber_record -U
```

Then you can reference `cyber_record` by
```python
from cyber_record.record import Record
```


## Examples
Below are some examples to help you read and write messages from record files.

## Read messages
You can read messages directly from the record file in the following ways.
```python
from cyber_record.record import Record

file_name = "20210521122747.record.00000"
record = Record(file_name)
for topic, message, t in record.read_messages():
  print("{}, {}, {}".format(topic, type(message), t))
```

The following is the output log of the program
```
/apollo/localization/pose, <class 'LocalizationEstimate'>, 1627031535246897752
/apollo/canbus/chassis, <class 'Chassis'>, 1627031535246913234
/apollo/canbus/chassis, <class 'Chassis'>, 1627031535253680838
```

#### Filter Read
You can also read messages filtered by topics and time. This will improve the speed of parsing messages.
```python
def read_filter_by_both():
  record = Record(file_name)
  for topic, message, t in record.read_messages('/apollo/canbus/chassis', \
      start_time=1627031535164278940, end_time=1627031535215164773):
    print("{}, {}, {}".format(topic, type(message), t))
```


## Parse messages
To avoid introducing too many dependencies, you can save messages by `record_msg`.
```
pip install record_msg
```

`record_msg` provides 3 types of interfaces

#### csv format
you can use `to_csv` to format objects so that they can be easily saved in csv format.
```python
f = open("message.csv", 'w')
writer = csv.writer(f)

def parse_pose(pose):
  '''
  save pose to csv file
  '''
  line = to_csv([pose.header.timestamp_sec, pose.pose])
  writer.writerow(line)

f.close()
```

#### image
you can use `ImageParser` to parse and save images.
```python
image_parser = ImageParser(output_path='../test')
for topic, message, t in record.read_messages():
  if topic == "/apollo/sensor/camera/front_6mm/image":
    image_parser.parse(message)
    # or use timestamp as image file name
    # image_parser.parse(image, t)
```

#### lidar
you can use `PointCloudParser` to parse and save pointclouds.
```python
pointcloud_parser = PointCloudParser('../test')
for topic, message, t in record.read_messages():
  if topic == "/apollo/sensor/lidar32/compensator/PointCloud2":
    pointcloud_parser.parse(message)
    # other modes, default is 'ascii'
    # pointcloud_parser.parse(message, mode='binary')
    # pointcloud_parser.parse(message, mode='binary_compressed')
```


## Write messages
You can now also build record by messages.
```python
def write_message():
  pb_map = map_pb2.Map()
  pb_map.header.version = 'hello'.encode()

  with Record(write_file_name, mode='w') as record:
    record.write('/apollo/map', pb_map, int(time.time() * 1e9))
```

Its application scenario is to convert dataset into record files. Please note that it must be written in chronological order.
