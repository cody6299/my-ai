## Tasks

- [x] Task 1：新增 Entity（cloud-future-depositwithdraw/future-depositwithdraw-dal/.../entity/InternalTransferWithdrawLimitRecordEntity.java）
- [x] Task 2：新增 Mapper（cloud-future-depositwithdraw/future-depositwithdraw-dal/.../mapper/InternalTransferWithdrawLimitRecordMapper.java）
- [x] Task 3：新增 Kafka 消息 DTO（cloud-future-depositwithdraw/future-depositwithdraw-common/.../model/notify/kafka/AssetTransferMessage.java）
- [x] Task 4：修改 KafkaConfig，新增 assetTransferListenerContainerFactory（cloud-future-depositwithdraw/future-depositwithdraw-common/.../config/KafkaConfig.java）
- [x] Task 5：新增 AssetTransferListenService，实现 Kafka 消费 + 事务处理（cloud-future-depositwithdraw/future-depositwithdraw-common/.../service/AssetTransferListenService.java）
