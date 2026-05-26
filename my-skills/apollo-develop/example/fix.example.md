## AP-10677 问题修复

## 问题描述
getPosition接口返回的positionSide不对，因为计算过程中没有考虑到shortQty < 0的事实，造成了更新仓位方向错误的问题。

## 涉及服务

- [cloud-future-depositwithdraw](https://git.toolsapple.net/ApolloX/cloud-future-streamer/commit/567ded8af6e9b2bb912a0f7bd5cc0cfda0cc42dd)

## 处理结果

- [cloud-future-streamer](https://git.toolsapple.net/ApolloX/cloud-future-streamer/commit/567ded8af6e9b2bb912a0f7bd5cc0cfda0cc42dd)

    - PositionService[修改]: 修改了仓位方向的相关问题
