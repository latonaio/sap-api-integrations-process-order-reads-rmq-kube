# sap-api-integrations-process-order-reads-rmq-kube  
sap-api-integrations-process-order-reads-rmq-kube は、外部システム(特にエッジコンピューティング環境)をSAPと統合することを目的に、SAP API でプロセス指図データを取得するマイクロサービスです。    
sap-api-integrations-process-order-reads-rmq-kube には、サンプルのAPI Json フォーマットが含まれています。   
sap-api-integrations-process-order-reads-rmq-kube は、オンプレミス版である（＝クラウド版ではない）SAPS4HANA API の利用を前提としています。クラウド版APIを利用する場合は、ご注意ください。   
https://api.sap.com/api/OP_API_PROCESS_ORDER_2_SRV_0001/overview   

## 動作環境  

sap-api-integrations-process-order-reads-rmq-kube は、主にエッジコンピューティング環境における動作にフォーカスしています。  
使用する際は、事前に下記の通り エッジコンピューティングの動作環境（推奨/必須）を用意してください。  
・ エッジ Kubernetes （推奨）    
・ AION のリソース （推奨)    
・ OS: LinuxOS （必須）    
・ CPU: ARM/AMD/Intel（いずれか必須）    
・ RabbitMQ on Kubernetes  
・ RabbitMQ Client  

## クラウド環境での利用

sap-api-integrations-process-order-reads-rmq-kube は、外部システムがクラウド環境である場合にSAPと統合するときにおいても、利用可能なように設計されています。  

## RabbitMQ からの JSON Input

sap-api-integrations-process-order-reads-rmq-kube は、Inputとして、RabbitMQ からのメッセージをJSON形式で受け取ります。 
Input の サンプルJSON は、Inputs フォルダ内にあります。  

## RabbitMQ からのメッセージ受信による イベントドリヴン の ランタイム実行

sap-api-integrations-process-order-reads-rmq-kube は、RabbitMQ からのメッセージを受け取ると、イベントドリヴンでランタイムを実行します。  
AION の仕様では、Kubernetes 上 の 当該マイクロサービスPod は 立ち上がったまま待機状態で当該メッセージを受け取り、（コンテナ起動などの段取時間をカットして）即座にランタイムを実行します。　

## RabbitMQ への JSON Output

sap-api-integrations-process-order-reads-rmq-kube は、Outputとして、RabbitMQ へのメッセージをJSON形式で出力します。  
Output の サンプルJSON は、Outputs フォルダ内にあります。  

## RabbitMQ の マスタサーバ環境

sap-api-integrations-process-order-rmq-kube が利用する RabbitMQ のマスタサーバ環境は、[rabbitmq-on-kubernetes](https://github.com/latonaio/rabbitmq-on-kubernetes) です。  
当該マスタサーバ環境は、同じエッジコンピューティングデバイスに配置されても、別の物理(仮想)サーバ内に配置されても、どちらでも構いません。

## RabbitMQ の Golang Runtime ライブラリ
sap-api-integrations-process-order-reads-rmq-kube は、RabbitMQ の Golang Runtime ライブラリ として、[rabbitmq-golang-client](https://github.com/latonaio/rabbitmq-golang-client)を利用しています。

## デプロイ・稼働
sap-api-integrations-process-order-reads-rmq-kube の デプロイ・稼働 を行うためには、aion-service-definitions の services.yml に、本レポジトリの services.yml を設定する必要があります。

kubectl apply - f 等で Deployment作成後、以下のコマンドで Pod が正しく生成されていることを確認してください。
```
$ kubectl get pods
```

## 本レポジトリ が 対応する API サービス
sap-api-integrations-process-order-reads-rmq-kube が対応する APIサービス は、次のものです。

* APIサービス概要説明 URL: https://api.sap.com/api/OP_API_PROCESS_ORDER_2_SRV_0001/overview
* APIサービス名(=baseURL): API_PROCESS_ORDER_2_SRV

## 本レポジトリ に 含まれる API名
sap-api-integrations-process-order-reads-rmq-kube には、次の API をコールするためのリソースが含まれています。  

* A_ProcessOrder_2（プロセス指図 - 一般）※製造指図の詳細データを取得するために、ToProcessOrderComponent、ToProcessOrderItem、ToProcessOrderOperation、ToProcessOrderStatus、と合わせて利用されます。
* ToProcessOrderComponent（プロセス指図 - 構成品目）
* ToProcessOrderItem（プロセス指図 - 明細）
* ToProcessOrderOperation（プロセス指図 - 作業手順）
* ToProcessOrderStatus（プロセス指図 - ステータス）

## API への 値入力条件 の 初期値
sap-api-integrations-process-order-reads-rmq-kube において、API への値入力条件の初期値は、入力ファイルレイアウトの種別毎に、次の通りとなっています。  

### SDC レイアウト

* inoutSDC.ProcessOrder.ManufacturingOrder（プロセス指図）

## SAP API Bussiness Hub の API の選択的コール

Latona および AION の SAP 関連リソースでは、Inputs フォルダ下の sample.json の accepter に取得したいデータの種別（＝APIの種別）を入力し、指定することができます。  
なお、同 accepter にAll(もしくは空白)の値を入力することで、全データ（＝全APIの種別）をまとめて取得することができます。  

* sample.jsonの記載例(1)  

accepter において 下記の例のように、データの種別（＝APIの種別）を指定します。  
ここでは、"General" が指定されています。    
  
```
	"api_schema": "sap.s4.beh.processorder.v1.ProcessOrder.Created.v1",
	"accepter": ["General"],
	"process_order": "1003463",
	"deleted": false
```
  
* 全データを取得する際のsample.jsonの記載例(2)  

全データを取得する場合、sample.json は以下のように記載します。  

```
	"api_schema": "sap.s4.beh.processorder.v1.ProcessOrder.Created.v1",
	"accepter": ["All"],
	"process_order": "1003463",
	"deleted": false
```



## 指定されたデータ種別のコール

accepter における データ種別 の指定に基づいて SAP_API_Caller 内の caller.go で API がコールされます。  
caller.go の func() 毎 の 以下の箇所が、指定された API をコールするソースコードです。  

```
func (c *SAPAPICaller) AsyncGetProcessOrder(manufacturingOrder string, accepter []string) {
	wg := &sync.WaitGroup{}
	wg.Add(len(accepter))
	for _, fn := range accepter {
		switch fn {
		case "General":
			func() {
				c.General(manufacturingOrder)
				wg.Done()
			}()
		default:
			wg.Done()
		}
	}

	wg.Wait()
}
```

## Output  
本マイクロサービスでは、[golang-logging-library-for-sap](https://github.com/latonaio/golang-logging-library-for-sap) により、以下のようなデータがJSON形式で出力されます。  
以下の sample.json の例は、SAP プロセス指図 の 一般データ が取得された結果の JSON の例です。  
以下の項目のうち、"ManufacturingOrder" ～ "to_ProcessOrderStatus" は、/SAP_API_Output_Formatter/type.go 内 の Type General {} による出力結果です。"cursor" ～ "time"は、golang-logging-library-for-sap による 定型フォーマットの出力結果です。  

```
{
	"cursor": "/Users/latona2/bitbucket/sap-api-integrations-process-order-reads-rmq-kube/SAP_API_Caller/caller.go#L53",
	"function": "sap-api-integrations-process-order-reads-rmq-kube/SAP_API_Caller.(*SAPAPICaller).General",
	"level": "INFO",
	"message": [
		{
			"ManufacturingOrder": "1003463",
			"ManufacturingOrderCategory": "40",
			"ManufacturingOrderType": "YBM2",
			"ManufacturingOrderImportance": "",
			"OrderIsCreated": "",
			"OrderIsReleased": "",
			"OrderIsPrinted": "",
			"OrderIsConfirmed": "",
			"OrderIsPartiallyConfirmed": "",
			"OrderIsDelivered": "",
			"OrderIsDeleted": "",
			"OrderIsPreCosted": "X",
			"SettlementRuleIsCreated": "X",
			"OrderIsPartiallyReleased": "",
			"OrderIsLocked": "",
			"OrderIsTechnicallyCompleted": "X",
			"OrderIsClosed": "",
			"OrderIsPartiallyDelivered": "",
			"OrderIsMarkedForDeletion": "",
			"SettlementRuleIsCrtedManually": "",
			"OrderIsScheduled": "",
			"OrderHasGeneratedOperations": "",
			"OrderIsToBeHandledInBatches": "X",
			"MaterialAvailyIsNotChecked": "X",
			"MfgOrderCreationDate": "2017-08-30T09:00:00+09:00",
			"MfgOrderCreationTime": "PT09H05M40S",
			"LastChangeDateTime": "20200723125734",
			"Material": "FG29",
			"StorageLocation": "171A",
			"GoodsRecipientName": "",
			"UnloadingPointName": "",
			"InventoryUsabilityCode": "",
			"MaterialGoodsReceiptDuration": "0",
			"QuantityDistributionKey": "",
			"StockSegment": "",
			"OrderInternalBillOfOperations": "3508",
			"ProductionPlant": "1710",
			"Plant": "1710",
			"MRPArea": "1710",
			"MRPController": "001",
			"ProductionSupervisor": "YB2",
			"ProductionVersion": "0001",
			"PlannedOrder": "491",
			"SalesOrder": "",
			"SalesOrderItem": "0",
			"BasicSchedulingType": "1",
			"ManufacturingObject": "OR000001003463",
			"ProductConfiguration": "0",
			"OrderSequenceNumber": "0",
			"BusinessArea": "",
			"CompanyCode": "1710",
			"ProfitCenter": "YB110",
			"ActualCostsCostingVariant": "PYG2",
			"PlannedCostsCostingVariant": "PYG1",
			"FunctionalArea": "",
			"MfgOrderPlannedStartDate": "2017-08-30T09:00:00+09:00",
			"MfgOrderPlannedStartTime": "PT00H00M00S",
			"MfgOrderPlannedEndDate": "2017-09-06T09:00:00+09:00",
			"MfgOrderPlannedEndTime": "PT00H00M00S",
			"MfgOrderScheduledStartDate": "2017-09-01T09:00:00+09:00",
			"MfgOrderScheduledStartTime": "PT06H00M00S",
			"MfgOrderScheduledEndDate": "2017-09-01T09:00:00+09:00",
			"MfgOrderScheduledEndTime": "PT06H11M38S",
			"MfgOrderActualReleaseDate": "2017-08-30T09:00:00+09:00",
			"ProductionUnit": "BT",
			"TotalQuantity": "969",
			"MfgOrderPlannedScrapQty": "0",
			"MfgOrderConfirmedYieldQty": "0",
			"CustomerName": "",
			"WBSElementExternalID": "",
			"OrderLongText": "",
			"to_ProcessOrderComponent": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_PROCESS_ORDER_2_SRV/A_ProcessOrder_2('1003463')/to_ProcessOrderComponent",
			"to_ProcessOrderItem": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_PROCESS_ORDER_2_SRV/A_ProcessOrder_2('1003463')/to_ProcessOrderItem",
			"to_ProcessOrderOperation": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_PROCESS_ORDER_2_SRV/A_ProcessOrder_2('1003463')/to_ProcessOrderOperation",
			"to_ProcessOrderStatus": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_PROCESS_ORDER_2_SRV/A_ProcessOrder_2('1003463')/to_ProcessOrderStatus"
		}
	],
	"time": "2022-01-28T16:17:24+09:00"
}
```
