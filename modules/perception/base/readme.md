<a name="perception_base" />

## base

### blob
定义了一个名为Blob的C++类，表示一个多维数组。该类具有各种成员函数来操作数组，例如调整大小、重新定义形状、访问元素和从另一个Blob复制数据等。Blob类使用模板来允许使用不同的数据类型（Dtype），并且还支持GPU内存。该类具有一个名为data_的成员变量，类型为std::shared_ptr<SyncedMemory>，用于保存Blob的实际数据。Blob类旨在用于神经网络框架或类似应用程序的上下文中。

### object
