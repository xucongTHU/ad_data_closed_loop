# ad_data_closed_loop

## architecture
```
|-- ad_data_closed_loop
    |-- config
    |-- docker
    |-- docs
        |-- new_features_cn.md
        |-- new_featrures.md
    |-- infra
        |-- data_collector  //数据采集模块，使用ad_shadowmode<repo_url>
        |-- data_processor  //数据处理模块
            |-- opt
            |-- utils
            |-- visualizer
        |-- navigation_planner  //数采路径规划模块
            |-- nav_planner_node.cpp
            |-- nav_planner_node.h
            |-- utils
                |-- planner_utils.h
                |-- planner_utils.cpp
            |-- config
                |-- planner_weights.yaml
            |-- rl
                |-- planner_optRoute.cpp
                |-- planner_optRoute.h
                |-- planner_reward.cpp
                |-- planner_reward.h
    |-- training
        |-- planner_rl_train.py
        |-- ad_algorithm_train.py
        |-- embodied_ai_algorithm_train.py
        |-- settings.py
        |-- utils
        |-- datasets
        |-- models
        |-- benchmark
    |-- ops //运维部署
        |-- ad_data_closed_loop_deploy.sh
        |-- ad_algorithm_model_deploy.sh
        |-- embodied_ai_algorithm_model_deploy.sh
    |-- tests
    |-- tools
    |-- requirements.txt
    |-- README.md
```