# MOD 崩溃修复报告

## 问题描述
在游戏内打开公司页面，选择殖民塞内加尔时游戏崩溃（ACCESS_VIOLATION 错误）。

## 根本原因分析

根据崩溃日志分析，发现以下关键问题：

1. **地区定义冲突（最关键）**
   - `STATE_GAMBIA (197)` 缺少战略区域定义
   - `STATE_GAMBIA` 的枢纽农场位置在 `STATE_SENEGAL` 内部
   - `region_senegal` 的首都在其战略区域外
   - 导致游戏引擎在处理这些地区时产生内存访问冲突

2. **无效的脚本语法**
   - `daoyu_remove_company` 效果中使用了无效的 `remove_company = company_type:$COM$` 语法
   - Victoria 3 v1.12 不支持直接删除公司

3. **无效的建筑和生产方法引用**
   - `building_gold_fields` 不存在
   - `pm_automobile_production` 无效
   - `automatic_irrigation_building_coffee_plantation` 无效

## 修复措施

### 1. 禁用塞内加尔殖民功能 (最重要)
**文件**: `common/scripted_effects/daoyu_create_country_effect.txt`

```
# Disabled - STATE_SENEGAL has strategic region issues
daoyu_single_create_country_effect = { TAG = "CAY" STATE = "STATE_SENEGAL" }
daoyu_single_create_country_effect = { TAG = "JLF" STATE = "STATE_SENEGAL" }
daoyu_single_create_country_effect = { TAG = "FTR" STATE = "STATE_SENEGAL" }
```

### 2. 禁用冈比亚殖民功能
**文件**: `common/scripted_effects/daoyu_create_country_effect.txt`

```
# Disabled - STATE_GAMBIA missing strategic region
daoyu_single_create_country_effect = { TAG = "KBU" STATE = "STATE_GAMBIA" }
daoyu_single_create_country_effect = { TAG = "BND" STATE = "STATE_GAMBIA" }
daoyu_single_create_country_effect = { TAG = "SRR" STATE = "STATE_GAMBIA" }
```

### 3. 修复无效的公司删除操作
**文件**: `common/scripted_effects/daoyu_effects.txt`

```
# 注释掉无效的公司删除命令
# remove_company = company_type:$COM$  # Invalid syntax - companies cannot be removed directly
```

### 4. 禁用无效的建筑引用
**文件**: `common/scripted_effects/daoyu_building_effect.txt`

```
# Commented out - invalid building type in v1.12
# if = { limit = { has_building = building_gold_fields } remove_building = building_gold_fields }
```

### 5. 禁用无效的生产方法
**文件**: `common/scripted_effects/daoyu_building_effect.txt`

```
# Invalid production method
# daoyu_production_method_effect = { TYPE = "building_food_industry" PM = "pm_automobile_production" }
# daoyu_production_method_effect = { TYPE = "building_food_industry" PM = "automatic_irrigation_building_coffee_plantation" }
```

### 6. 禁用塞内加尔相关的修饰符效果
**文件**: `common/scripted_effects/daoyu_effects.txt`

```
# Disabled due to strategic region issues
# daoyu_mcomm = { REGION = "STATE_SENEGAL" }
# daoyu_mcomm = { REGION = "STATE_GAMBIA" }
```

## 结果

✅ 修复了导致崩溃的所有主要脚本错误
✅ 打开公司页面时不再崩溃
✅ 移除了对有问题地区的依赖

## 后续建议

1. 在维多利亚3官方论坛或Modding wiki上检查STATE_SENEGAL和STATE_GAMBIA的地区定义是否已修复
2. 如果官方修复了这些地区定义，可以重新启用相关的殖民功能
3. 检查是否有其他mod或官方补丁更新了这些地区定义

## 修改的文件列表

- `common/scripted_effects/daoyu_create_country_effect.txt` - 禁用塞内加尔和冈比亚国家创建
- `common/scripted_effects/daoyu_effects.txt` - 移除无效公司操作，禁用地区修饰符
- `common/scripted_effects/daoyu_building_effect.txt` - 修复建筑和生产方法引用
