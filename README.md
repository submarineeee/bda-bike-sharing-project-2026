# bda-bike-sharing-project-2026
your-repo/
├── README.md                    # 怎么运行、数据来源、产出说明（= 要求里的 readme.txt）
├── requirements.txt             # R 版本 + 所有包版本（session_info 生成）
├── report.ipynb                 # 主 notebook（完整流程，由 final-version 改名）
├── data/
│   ├── train.csv                # 比赛训练数据
│   ├── test.csv                 # 比赛测试数据
│   ├── weather_vienna.rds       # 缓存的 open-meteo 天气（外部数据）
│   ├── station_elevation.rds    # 缓存的海拔
│   └── gtfs/…                    # 用到的 GTFS feed（E1 假日判定）
├── output/
│   ├── submission.csv           # 模型1（a=3.09）——notebook 生成
│   └── submission_2.csv         # 模型2（可选，如 a=1 回退版）
└── figures/                     # 仅当 notebook 引用外部 PNG 时才需要
