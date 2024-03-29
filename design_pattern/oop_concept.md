# 物件導向程式設計Object-oriented programming(OOP) 概念

## 一個抽象
- 找出關鍵性特徵並加以描述
- 簡化模型以協助思考與運用
- 依賴抽象

## 兩個目標
- 高內聚
- 低耦合

## 三個特性
- 繼承
- 封裝
- 多型

## 六個原則 SOLID
### 單一職責員則 Single Responsibility Principle
+ 就一個類別而言，應該僅有一個引起它變化的原因
+ 一個模組應該只對唯一的一個角色負責

### 開閉原則 Open-Closed Principle
+ 對擴展開放，對修改封閉
+ 新功能的添加應該是透過增加新的代碼或模組去達成，而非修改現有代碼，使得系統的穩定性得以保持。

### 里式替換原則 Liskov's Substitution Principle
+ 軟體使用父類別的地方，一定也會適用於子類別
+ 子類別應該能夠替換父類別而不影響程式的正確性

### 介面隔離原則 The Interface Segregation Principle
+ 客戶端不應該倚賴它不需要的介面
+ 類別間的倚賴應建立在最小的介面上

### 倚賴倒置原則 The Dependency Inversion Principle
+ 高層模組不應倚賴低層模組，兩者都應該倚賴抽象
+ 抽象不應該倚賴細節，細節應該倚賴抽象

### 最少知識原則(迪米特法則) Law of Demeter
+ 一個物件應該對其他物件有最少了解