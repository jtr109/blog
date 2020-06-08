---
title: Python gRPC Json Format 的小坑
date: 2019-11-07 22:31:12
tags:
  - python
  - gprc
  - json
typora-root-url: ../../static
---
## 背景

最近在开发一个网关项目, 需要将外部的 JSON 请求转换成 gRPC 请求向后转发, 并将 gRPC 响应转换为 JSON 格式的响应返回.

实现过程中发现使用 `google.protobuf.json_format.MessageToDict` 的时候 ( `google.protobuf.json_format.MessageToJson` 同理) 有一些需要注意留意的细节.

## 遇到问题

当使用 `MessageToDict`  对 messge 转换时, 发现两个不符合预期的问题:

- 如果一个 message field 对应的值为 zero value, 对应的 `dict` 中不会显示该键值对
- field 名称会从 snake_case 被转换成 lowerCamelCase

## 源码分析

以 `MessageToDict` 为例, 我们看一下函数的定义.

```python
def MessageToDict(
    message,
    including_default_value_fields=False,
    preserving_proto_field_name=False,
    use_integers_for_enums=False,
    descriptor_pool=None):
  """Converts protobuf message to a dictionary.

  When the dictionary is encoded to JSON, it conforms to proto3 JSON spec.

  Args:
    message: The protocol buffers message instance to serialize.
    including_default_value_fields: If True, singular primitive fields,
        repeated fields, and map fields will always be serialized.  If
        False, only serialize non-empty fields.  Singular message fields
        and oneof fields are not affected by this option.
    preserving_proto_field_name: If True, use the original proto field
        names as defined in the .proto file. If False, convert the field
        names to lowerCamelCase.
    use_integers_for_enums: If true, print integers instead of enum names.
    descriptor_pool: A Descriptor Pool for resolving types. If None use the
        default.

  Returns:
    A dict representation of the protocol buffer message.
  """
      # ...
```

所以自动忽略 zero value 和 field 转换分别是由 `including_default_value_fields` 和 `preserving_proto_field_name` 两个参数控制.

## 结论

在调用 `MessageToDict` 函数时, 将以上两个参数指定为 `True` 即可解决问题. [代码示例](https://github.com/jtr109/zero-value-in-protos/blob/master/example.ipynb)如下:

![](python-grpc-json-format/1570964227734-2317f63a-ba36-476a-8a80-58947a84aac4.png)

示例代码可以在[这个项目](https://github.com/jtr109/zero-value-in-protos)中查看

## 思考

这个问题会令人在意的主要原因是因为 Python 是一个弱类型语言, 或许 protobuf 在传输和存储过程中, 为了考虑性能, 确实会用省略字段的方式来表达零值, 就像[这个回答](https://github.com/protocolbuffers/protobuf/issues/2796#issuecomment-284872746)中说到的那样. 只要在将 protobuf 转换为特定语言的 struct 时再对应调整成恰当的结构就能满足需求, 但是在转换成 JSON 的时候默认将这个字段抹去, 我仍然认为是不合理的.

## Reference

- python grpc package tutorial: [https://grpc.io/docs/tutorials/basic/python/](https://grpc.io/docs/tutorials/basic/python/)
- 关于 zero value 默认忽略的解释: [https://github.com/protocolbuffers/protobuf/issues/2796](https://github.com/protocolbuffers/protobuf/issues/2796)
