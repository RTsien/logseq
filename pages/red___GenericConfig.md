- 例子
  ```yaml
  apiVersion: configurator.red.tencent.com/v1alpha1
  kind: GenericConfig
  metadata:
    name: traffic-config
  config:
    #  x-release-version的生成规则
    release_version:
      - name: ${{x_release_version}}   # x-release-version的值
        enable:bool
        # 匹配条件
        match:
          type: OR # 可选，默认为OR，表示任一匹配条件满足即可
          # 灰度的小区列表（可选）
          zone:
            zoneids: 
              - 20004
          # 灰度的玩家列表（可选）
          player:
            gids: 
              - 10003001
          # 最小客户端版本号（可选）
          version:
            min_client_version: 111
            min_res_version: 111
          # 灰度的服务
          server:
            releases:   # relasse-name
              - roomsvr-v1
              - infosvr-v1
  ```