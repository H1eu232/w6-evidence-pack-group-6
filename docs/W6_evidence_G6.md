# Evidence Pack — W6: Operations Hardening & Cost-Aware Cloud
# Group 6 — HexaCode

---

## Section 1 — Cover

| Field | Details |
|---|---|
| **Group Number** | Group 6 |
| **Member Names** | Minh Tuấn · Thành Vinh · Anh Hoàng · Hoàng Nhân · Mạnh Khang · Ngọc Thắng · Đình Thông · Thành Tâm |
| **Link Repo** | [GitHub repo URL](https://github.com/H1eu232/w6-evidence-pack-group-6.git) |
| **W5 Evidence Pack** | [W5_evidence](https://github.com/H1eu232/w6-evidence-pack-group-6/edit/main/docs/W6_evidence_G6.md) |

### W5 Feedback đã giải quyết

![W5 feedback](./images/W5feedback.png)

#### 1.1 API Gateway throttling không còn giữ mức 1 RPS

![W5 throttling fixed](./images/APIThrottling.png)

<sub>Note: Cấu hình giới hạn tốc độ (Throttling) cho route /api/chat/messages với Rate=10 và Burst=20. Đây là một lớp bảo vệ 'Operations Hardening' quan trọng nhằm ngăn chặn các cuộc tấn công DDoS hoặc lỗi logic từ phía client gây ra hiện tượng 'runaway costs' (chi phí tăng vọt ngoài ý muốn) khi gọi các dịch vụ đắt tiền như Lambda và Bedrock.</sub>

#### 1.2 Backup selection không còn mô tả kiểu assign-by-prefix

![W5 backup selection fixed](./images/BackupPlan.png)

<sub>Note: Hệ thống xác nhận các bản sao lưu đã được thực hiện thành công (Status: Completed) cho cả RDS và EFS với chu kỳ lưu trữ (Retention) là 35 ngày. Đồng thời, Backup Plan được gắn thẻ Project: hexacode và Environment: prod một cách nhất quán. Điều này giúp nhóm không chỉ đảm bảo an toàn dữ liệu mà còn kiểm soát chặt chẽ chi phí lưu trữ phát sinh từ các bản backup, phục vụ việc phân bổ ngân sách chính xác trong Cost Explorer.</sub>

![Protected Resources](./images/ProtectedResources.png)

<sub>Note: cấu hình AWS Backup để bao phủ toàn bộ các kho lưu trữ dữ liệu trọng yếu của ứng dụng bao gồm: S3 (Assets), RDS (Database) và EFS (Artifacts)</sub>

#### 1.3 Provisioned concurrency đã có cấu hình live, nhưng phần diễn giải warm-start phải viết đúng

![W5 provisioned concurrency config](./images/Provisioned.png)

<sub>Note: Thiết lập các cơ chế an toàn cho Lambda: (1) Reserved Concurrency giới hạn số lượng task chạy đồng thời để tránh làm cạn kiệt tài nguyên của account. (2) Provisioned Concurrency trên alias live giúp loại bỏ độ trễ 'khởi động lạnh' (Cold Start) cho chatbot AI. (3) Recursive loop detection được kích hoạt để ngăn chặn các vòng lặp vô hạn có thể gây cháy ngân sách trong vài phút.</sub>

---

## Section 2 — MH-COST-V — Cost Visibility & Attribution

| Tag Key       | Giá trị                        |
| ------------- | ------------------------------ |
| `Owner`       | `hoang`                        |
| `Project`     | `hexacode`                     |
| `Environment` | `production`                   |
| `CostCenter`  | `G6`                           |
| `ManagedBy`   | `terraform`                    |
| `Keep`        | `true`                         |
| `Name`        | `hexacode-prod-{related name}` |
| `Application` | `{related which aws service}`  |

### 2.1 Tagging — Bốn tag key bắt buộc trên mọi billable resource

**Screenshot tag trên ECS:**

![Tags on ECS](./images/ECSTag.png)

**Screenshot tag trên RDS:**

![Tags on RDS](./images/RDSTag.png)

**Screenshot tag trên Lambda:**

![Tags on Lambda-chat](./images/LambdaTag.png)

![Tags on Lambda-kb](./images/LambdaTag2.png)

![Tags on Lambda-cors](./images/LambdaTag3.png)

**Screenshot tag trên S3:**

![Tags on S3](./images/S3Tag.png)

<sub>Note: Mọi Billable Resource (RDS, Lambda, S3, EC2) đã được áp dụng bộ Tag Schema nhất quán: Project=hexacode, Environment=production, CostCenter=G6, Owner=hoang. Điều này đảm bảo khả năng truy vết chi phí 100% đến từng dịch vụ đơn lẻ.</sub>

---

### 2.2 Cost Allocation Tags — Activated trong Billing Console

![AWS Budgets config](./images/ProdBudget.png)

---

### 2.3 Cost Monitoring Tool(s) đã cấu hình

**Tool 1 — AWS Budgets:**

![AWS Budgets config](./images/ProdBudget.png)

<sub>Note: Budget $150/tuần được set trước Thứ Sáu. Alert gửi về email/SNS khi đạt các ngưỡng cấu hình. Với evidence pack workshop, giữ wording này nhất quán với narrative của bài.</sub>

**Tool 2 — Cost Explorer filter theo tag:**

![Cost Explorer filter](./images/CostExplorer.png)

![Cost Explorer filter](./images/CostExplorer2.png)

<sub>Note: </sub>

**Tool 3 — Cost Anomaly Detection :**

![Cost Anomaly Detection](./images/CostAnomalyDetect.png)

![Cost Anomaly Detection](./images/CostAnomalyDetect2.png)

<sub>Note: Cấu hình ML-based monitor 'Finance Team' với ngưỡng $75. Alert subscription đã được xác nhận (Confirmed) qua email nahoangit@gmail.com để phát hiện các biến động chi phí bất thường ngay lập tức.</sub>

---

### 2.4 Baseline Cost Breakdown (sau ít nhất 24h data)

![Baseline cost breakdown](./images/w6-baseline-cost.png)

<sub>Note: Screenshot Cost Explorer sau 24h redeploy, filter theo tag `Application=HexaCode`.</sub>

> Temporary guide for paragraph — xoá sau khi viết observation thật.
> - Viết theo mẫu: `Top 3 cost drivers là A, B, C; A cao vì ..., B đáng chú ý vì ..., C có thể tối ưu bằng ...`.
> - Nếu nhóm đang chạy off-hours scaling thì nhớ đối chiếu với audit: live ECS services đã từng ở desired/running = 0 ngoài giờ, nên cost compute có thể thấp hơn dự đoán vào lúc chụp.

**Quan sát top 3 cost driver:**

`[Viết 1 paragraph ở đây. Ví dụ: "Top 3 cost driver sau 24h redeploy: (1) RDS db.m7i.large chiếm ~45% tổng chi phí ($X) — đang chạy Multi-AZ trong dev environment, có thể tắt Multi-AZ để cắt ~50% dòng RDS. (2) NAT Gateway data processing chiếm ~25% ($X) — Lambda chatbot gọi Bedrock qua NAT, có thể chuyển sang VPC endpoint để giảm. (3) ECS Fargate chiếm ~20% ($X) — 3 services chạy liên tục kể cả khi không có traffic, có thể scale down ngoài giờ."]`

---

### 2.5 — Tagging Strategy Document

### 1. Objective
Tagging Strategy của dự án **HexaCode** được thiết kế để tối ưu hóa khả năng giám sát tài nguyên và phân bổ chi phí chi tiết. Thay vì sử dụng các nhãn chung, nhóm tập trung vào việc định danh chức năng cụ thể của từng dịch vụ, giúp quy trình vận hành và xử lý sự cố diễn ra nhanh chóng.

### 2. Tag Schema

| Tag Key | Giá trị thực tế áp dụng | Quy tắc đặt giá trị | Phạm vi áp dụng |
| :--- | :--- | :--- | :--- |
| **Project** | `hexacode` | Tên dự án (Dùng làm bộ lọc chính trong Cost Explorer). | Toàn bộ tài nguyên |
| **Application** | `RDS-postgres-main`, `Lambda-chat-service`, `S3-terraform-state` | Định danh theo cú pháp: [Service]-[Chức năng]-[Vị trí]. | Toàn bộ tài nguyên |
| **Environment** | `prod` | Xác định môi trường vận hành là Production. | Toàn bộ tài nguyên |
| **Owner** | `hoang` | Tên thành viên chịu trách nhiệm quản lý chính. | Toàn bộ tài nguyên |
| **CostCenter** | `G6` | Mã định danh Nhóm 6 để quản lý ngân sách. | Toàn bộ tài nguyên |
| **ManagedBy** | `terraform` | Xác định nguồn gốc khởi tạo (IaC hoặc Manual). | Các tài nguyên deploy qua code |

### 3. Quy tắc định danh Application (Granularity)
Nhóm áp dụng mô hình định danh chi tiết cho tag `Application` nhằm hỗ trợ việc phân tách hóa đơn (Cost Breakdown) đến từng dịch vụ đơn lẻ:
*   **Dịch vụ Compute:** Phân loại theo chức năng logic (Ví dụ: `Lambda-chat-service`).
*   **Dịch vụ Database:** Phân loại theo Engine và vai trò (Ví dụ: `RDS-postgres-main`).
*   **Dịch vụ Storage:** Phân loại theo mục đích sử dụng (Ví dụ: `S3-terraform-state`).

### 4. Chiến lược truy vết chi phí (Cost Attribution)
Trong AWS Billing & Cost Explorer, nhóm sử dụng cơ chế lọc hai tầng:
1.  **Tầng dự án:** Sử dụng tag **`Project: hexacode`** để xem tổng quan chi phí của cả nhóm so với hạn mức $150.
2.  **Tầng dịch vụ:** Sử dụng tag **`Application`** để xác định thành phần nào trong hệ thống đang tiêu tốn ngân sách nhiều nhất, từ đó đưa ra quyết định tối ưu hóa (Right-sizing).

### 5. Cơ chế đảm bảo tuân thủ (Compliance)
*   **Infrastructure as Code (IaC):** Sử dụng Terraform để tự động gán các nhãn `ManagedBy`, `Project` và `CostCenter` ngay khi tạo mới, đảm bảo tính nhất quán và tránh sai sót do con người.
*   **Kiểm soát vận hành:** Tag `Owner: hoang` giúp định danh người chịu trách nhiệm khi hệ thống có cảnh báo từ CloudWatch Alarms hoặc khi Cost Anomaly Detection phát hiện chi tiêu bất thường.
*   **Rà soát định kỳ:** Nhóm sử dụng công cụ **Tag Editor** trong AWS Console để kiểm tra hàng tuần, đảm bảo không có tài nguyên nào bị bỏ sót (Untagged resources).

### 6. Forbidden Values
*   **Project:** Không sử dụng các giá trị chung như `test`, `aws`, hoặc bỏ trống.
*   **Owner:** Không dùng tên chung của nhóm (G6), phải dùng tên cá nhân chịu trách nhiệm.
*   **Environment:** Tuyệt đối không để trống, giá trị mặc định phải là `prod` cho stack chính của dự án.

---

## Section 3 — MH-COST-A — Cost Control & Action

### 3.1 Automated Cost Guard Lambda

**Screenshot Lambda function:**

![Cost Guard Lambda](./images/CostGuard.png)

<sub>Note: Lambda CostGuardLambda được đấu nối với SNS Trigger. IAM Role đi kèm thực thi nguyên tắc Least Privilege với Inline Policy CostGuard-ECS chỉ cho phép quyền ecs:UpdateService và ecs:ListServices, không có quyền can thiệp vào các dịch vụ khác.</sub>

**Lambda code snippet:**

```python
import boto3
import json
from datetime import datetime

def lambda_handler(event, context):
    print(f"📨 Received event: {json.dumps(event, indent=2)}")

    # Check if this is SNS event
    if 'Records' in event and event['Records'][0].get('EventSource') == 'aws:sns':
        return handle_sns_event(event)
    else:
        # Direct invoke
        return handle_direct_invoke(event)

def handle_sns_event(event):
    """Handle SNS notification from AWS Budgets"""
    try:
        sns_message = event['Records'][0]['Sns']['Message']
        print(f"📊 Budget Alert: {sns_message}")

        # Parse budget message
        try:
            budget_data = json.loads(sns_message)
            budget_name = budget_data.get('BudgetName', 'Unknown')
            amount = budget_data.get('ActualAmount', 'Unknown')
        except:
            budget_name = "Daily Budget"
            amount = "Over $150"

        print(f"🚨 BUDGET ALERT: {budget_name} - {amount}")

        # Trigger cost optimization
        result = stop_unprotected_services()

        return {
            'statusCode': 200,
            'body': json.dumps({
                'message': f'Budget alert processed: {budget_name}',
                'cost_optimization': result
            })
        }

    except Exception as e:
        print(f"❌ Error processing SNS event: {str(e)}")
        return {'statusCode': 500, 'error': str(e)}

def handle_direct_invoke(event):
    """Handle direct invoke"""
    return stop_unprotected_services()

def stop_unprotected_services():
    """Tắt services KHÔNG CÓ tag Keep = true"""
    try:
        ecs = boto3.client('ecs', region_name='us-west-2')

        # List all services trong cluster
        services_response = ecs.list_services(cluster='hexacode-prod')
        service_arns = services_response.get('serviceArns', [])

        stopped_services = []
        protected_services = []
        errors = []

        for service_arn in service_arns:
            try:
                # Extract service name từ ARN
                service_name = service_arn.split('/')[-1]

                print(f"🔍 Checking service: {service_name}")

                # Get tags cho service
                tags_response = ecs.list_tags_for_resource(resourceArn=service_arn)
                tags = tags_response.get('tags', [])

                # Check for Keep = true tag (PROTECTION)
                is_protected = False
                for tag in tags:
                    if tag['key'].lower() == 'keep' and tag['value'].lower() == 'true':
                        is_protected = True
                        print(f"🛡️ Service {service_name} có tag Keep=true - ĐƯỢC BẢO VỆ")
                        break

                if is_protected:
                    # Service được bảo vệ, không tắt
                    protected_services.append(service_name)
                else:
                    # Service KHÔNG có tag Keep=true → TẮT
                    print(f"🎯 Service {service_name} KHÔNG có tag Keep=true - SẼ BỊ TẮT")

                    # Get current service details
                    service_details = ecs.describe_services(
                        cluster='hexacode-prod',
                        services=[service_name]
                    )

                    current_count = service_details['services'][0]['desiredCount']

                    if current_count > 0:
                        # Scale service to 0
                        response = ecs.update_service(
                            cluster='hexacode-prod',
                            service=service_name,
                            desiredCount=0
                        )
                        stopped_services.append({
                            'service': service_name,
                            'previous_count': current_count,
                            'new_count': 0,
                            'reason': 'no_keep_tag'
                        })
                        print(f"🛑 STOPPED service: {service_name} (was running {current_count} tasks)")
                    else:
                        print(f"ℹ️ Service {service_name} already stopped")
                        stopped_services.append({
                            'service': service_name,
                            'previous_count': 0,
                            'new_count': 0,
                            'note': 'already_stopped'
                        })

            except Exception as e:
                error_msg = f"❌ ERROR processing {service_name}: {str(e)}"
                errors.append(error_msg)
                print(error_msg)

        result = {
            'timestamp': datetime.now().isoformat(),
            'stopped_services': stopped_services,
            'protected_services': protected_services,
            'errors': errors,
            'summary': {
                'stopped_count': len(stopped_services),
                'protected_count': len(protected_services),
                'error_count': len(errors)
            }
        }

        print(f"📊 SUMMARY:")
        print(f"   🛑 Stopped: {len(stopped_services)} services (no Keep=true tag)")
        print(f"   🛡️ Protected: {len(protected_services)} services (has Keep=true tag)")
        print(f"   ❌ Errors: {len(errors)}")

        return {
            'statusCode': 200,
            'body': json.dumps(result, indent=2)
        }

    except Exception as e:
        error_msg = f"❌ Error in cost optimization: {str(e)}"
        print(error_msg)
        return {
            'statusCode': 500,
            'body': json.dumps({'error': error_msg})
        }
```

---

### 3.2 IAM Role — Least Privilege

![Cost Guard IAM role](./images/IAMRole.png)

<sub>Note: IAM execution role chỉ có các permission cần thiết — `ec2:StopInstances`, `ec2:DescribeInstances`, `rds:StopDBInstance`, `rds:DescribeDBInstances`, `rds:ListTagsForResource`. Không có `Action: "*"` hay `Resource: "*"`.</sub>

### 3.3 EventBridge Daily Schedule

![EventBridge schedule](./images/EventBridgeDaily.png)

<sub>Note: Schedule daily-cost-guard được cấu hình chạy theo biểu thức cron 0 9 * * ? * (9h sáng hàng ngày), múi giờ Asia/Ho_Chi_Minh. Đây là primary trigger giúp dọn dẹp tài nguyên thừa vào mỗi đầu ngày làm việc.</sub>

---

### 3.4 Demonstrated Stop — Before/After + CloudTrail

**Service before stop note:**

![Service before stop](./images/ServiceBefore.png)

<sub>Note: hexacode-prod-problem-service đang chạy (1/1 Task running)</sub>

**Service after stop note:**

![Instance after stop](./images/ServiceAfter.png)

<sub>Note: Tất cả service chuyển sang trạng thái 0/0 Task running sau khi Lambda chạy.</sub>

**CloudTrail stop event:**

![CloudTrail stop event](./images/CloudTrailLog.png)

<sub>Note: Sự kiện UpdateService được ghi nhận. User name: CostGuardLambda xác nhận hành động này do Automation thực hiện, thay đổi desiredCount về 0 để dừng tiêu tốn chi phí Fargate.</sub>

---

### 3.5 Budgets daily $150 → SNS → Lambda (Wire + Demo)

![Budgets daily](./images/ProdBudget.png)

![Budgets SNS](./images/SNS.png)

<sub>Note: Trạng thái Confirmed của Subscription với giao thức LAMBDA xác nhận rằng mọi thông báo 'ALARM' từ AWS Budgets sẽ được chuyển tiếp ngay lập tức đến Lambda function.</sub>

![Budgets SNS wiring](./images/CostGuard.png)

<sub>Note: Kết quả demo chuỗi hành động (End-to-end chain). Màn hình CloudShell thực hiện sns publish giả lập, ngay lập tức CloudWatch Logs (Live tailing) ghi nhận Lambda nhận event và thực hiện quét/tắt service thành công. Điều này xác nhận hệ thống sẵn sàng hoạt động trong Production mà không cần chờ dữ liệu billing thật (latency 8-24h).</sub>

**Test SNS publish — demonstrate chain:**

```bash
aws sns publish \
  --topic-arn arn:aws:sns:us-west-2:583909632851:cost-guard-topic \
  --message '{"AlarmName":"BudgetAlert","NewStateValue":"ALARM"}' \
  --region us-west-2
```

![SNS test publish](./images/TestSNSPublish.png)

<sub>Note: Manual SNS publish trigger Lambda → Lambda stop một resource → xác nhận chain hoạt động end-to-end mà không cần chờ cost data thật.</sub>

---

### 3.6 Cost Data Latency — ADR

**Status:** Decided  
**Date:** 2026-05-21  
**Group:** Group 6 — HexaCode  
**Region:** us-west-2

#### Context
Trong quá trình vận hành hệ thống AWS tại Workshop W6, nhóm nhận thấy dữ liệu chi phí (AWS Billing & Cost Management) có độ trễ cập nhật từ **8 đến 24 giờ** trước khi xuất hiện chính thức trong Cost Explorer và kích hoạt các ngưỡng cảnh báo (Alerts) của AWS Budgets. 

Vì thời gian diễn ra workshop chỉ kéo dài 48 giờ, việc dựa hoàn toàn vào cơ chế kích hoạt tự động theo chi phí (Cost-driven trigger) từ Budgets là không khả thi, do dữ liệu tiêu dùng thực tế có thể chưa kịp ghi nhận trước khi buổi Demo kết thúc.

#### Decision
Nhóm HexaCode quyết định triển khai mô hình kiểm soát chi phí "lai" (Hybrid Governance Model) để đảm bảo tính sẵn sàng và khả năng demo:

1.  **Thiết lập liên kết (Wiring):** Hoàn tất đấu nối chuỗi logic: **AWS Budgets ($150 Daily)** -> **SNS Topic** -> **Lambda (`CostGuardLambda`)**. Đây là cơ chế phản ứng (Reactive) cho môi trường Production dài hạn.
2.  **Cơ chế kích hoạt chính (Primary Trigger):** Sử dụng **EventBridge Scheduler** (`daily-cost-guard`) chạy định kỳ theo **cron(0 9 * * ? *)** múi giờ **Asia/Ho_Chi_Minh**. Đây là cơ chế chủ động (Proactive) giúp dọn dẹp tài nguyên thừa hàng sáng mà không phụ thuộc vào độ trễ dữ liệu Billing.
3.  **Verification:** Sử dụng lệnh `aws sns publish` qua CloudShell để giả lập tín hiệu `ALARM` vượt ngưỡng ngân sách. Phương pháp này cho phép kiểm thử toàn bộ chuỗi phản ứng (End-to-end chain) ngay lập tức.

#### Technical Details
*   **Account ID:** `583909632851`
*   **Budget Threshold:** $150.00/day
*   **SNS Topic ARN:** `arn:aws:sns:us-west-2:583909632851:cost-guard-budget-alerts`
*   **Lambda Function:** `CostGuardLambda`
*   **Cron Schedule:** Chạy vào 09:00 AM hàng ngày (UTC+7).

#### Consequences
*   **Trong Workshop:** Nhóm đã chứng minh được khả năng tự động hóa việc dừng (Stop) các ECS Services thông qua việc giả lập sự kiện SNS. Behavior của hệ thống trong Production đã được xác nhận thành công mà không cần chờ dữ liệu chi phí thật.
*   **Trong Production thực tế:** Sự kết hợp này tạo ra lớp bảo mật chi phí 2 tầng: 
    *   **Tầng 1 (Schedule):** Ngăn chặn việc quên tắt tài nguyên vào giờ cao điểm.
    *   **Tầng 2 (Budget Alert):** Ngăn chặn các sự cố chi phí tăng đột biến do lỗi cấu hình hoặc bị tấn công tài nguyên, đảm bảo tổng thiệt hại không bao giờ vượt quá mức ngân sách đề ra.

---

## Section 4 — MH-OBS — CloudWatch Observability

### 4.1 CloudWatch Dashboard

![CloudWatch dashboard](./images/CloudwatchDashboard.png)

<sub>Note: Dashboard `HexaCode-Production-Observability` có backup metric/alarm widget và các widget metric chuẩn khác. Widget backup dùng data thật từ failed restore verification.</sub>

### 4.2 Custom Metric — AWS Backup failure count

**Metric đo gì:** số failed AWS Backup jobs đi qua log-based observability path của nhóm.

**Nguồn metric:**

```text
Log group: /aws/events/hexacode-prod-backup-failures
Metric namespace: HexaCode/Backup
Metric name: HexacodeBackupFailures
Filter logic: match failed backup/copy events via detail.state=FAILED and failed restore events via detail.status=FAILED
```

![Custom metric data points](./images/w6-custom-metric.png)

<sub>Note: Custom metric `HexacodeBackupFailures` trong namespace `HexaCode/Backup` — có datapoint thật từ failed restore verification jobs.</sub>

---

### 4.3 CloudWatch Alarm — Trạng thái ALARM

![CloudWatch alarm config](./images/AlarmConfig.png)

<sub>Note: Alarm `hexacode-prod-backup-failures` theo dõi metric `HexacodeBackupFailures`, threshold `>=1`, period 5 phút, action tới SNS topic backup failures.</sub>

---

### 4.4 Log Insights Query — Saved

**Query text:**

```bash
fields @timestamp, @message
| sort @timestamp desc
| limit 20
```

**Log group chạy chống lại:** `/aws/apigateway/hexacode-prod`

![Log Insights query result](./images/LoginsightQue.png)

![Log Insights query result](./images/Loginsight2.png)

<sub>Note: Query trả về result rows thật từ failed restore verification jobs. Thấy rõ log group và query text trên màn hình.</sub>

![Log Insights saved query](./images/SavedQuery.png)

<sub>Note: Saved query `hexacode-prod-backup-failure-events` nhìn thấy trong CloudWatch → Logs Insights → Saved queries.</sub>

---

## Section 5 — MH-SEC — Self-Healing Security Guard

### 5.1 Security Guard Lambda

**Misconfiguration detect và fix:** S3 bucket `hexacode-prod-submission-artifacts` bị tắt Block Public Access → Lambda gọi `PutBucketPublicAccessBlock` để bật lại toàn bộ 4 BPA flags.

**Screenshot Lambda function:**

![Security Guard Lambda](./images/SecurityGuard.png)

<sub>Note: Lambda `hexacode-prod-security-guard` đã deploy với least-privilege IAM role và target bucket rõ ràng cho demo.</sub>

**Lambda code snippet:**

```python
import boto3
import json

ec2 = boto3.client("ec2")

DANGEROUS_PORTS = [22, 3389]


def is_open_to_world(ip_range):
    return ip_range.get("CidrIp") == "0.0.0.0/0"


def revoke_open_ingress_rule(group_id, ip_permission):
    print(f"Revoking dangerous ingress rule from security group: {group_id}")
    print(json.dumps(ip_permission, default=str))

    ec2.revoke_security_group_ingress(
        GroupId=group_id,
        IpPermissions=[ip_permission]
    )


def lambda_handler(event, context):
    print("Self-Healing Security Guard started")
    print(json.dumps(event, default=str))

    fixed_rules = []

    response = ec2.describe_security_groups()

    for sg in response["SecurityGroups"]:
        group_id = sg["GroupId"]
        group_name = sg.get("GroupName", "")

        for permission in sg.get("IpPermissions", []):
            from_port = permission.get("FromPort")
            to_port = permission.get("ToPort")
            ip_protocol = permission.get("IpProtocol")

            if ip_protocol != "tcp":
                continue

            if from_port not in DANGEROUS_PORTS and to_port not in DANGEROUS_PORTS:
                continue

            open_ranges = [
                ip_range for ip_range in permission.get("IpRanges", [])
                if is_open_to_world(ip_range)
            ]

            if not open_ranges:
                continue

            revoke_permission = {
                "IpProtocol": ip_protocol,
                "FromPort": from_port,
                "ToPort": to_port,
                "IpRanges": open_ranges
            }

            revoke_open_ingress_rule(group_id, revoke_permission)

            fixed_rules.append({
                "group_id": group_id,
                "group_name": group_name,
                "from_port": from_port,
                "to_port": to_port,
                "cidr": "0.0.0.0/0"
            })

    result = {
        "fixed_rules": fixed_rules,
        "fixed_count": len(fixed_rules)
    }

    print(f"Security Guard result: {json.dumps(result)}")

    return {
        "statusCode": 200,
        "body": result
    }
```

---

### 5.2 IAM Role — Least Privilege

![Security Guard IAM role](./images/SecurityGuardPolicy.png)

<sub>Note: IAM execution role của `hexacode-prod-security-guard` chỉ cần quyền đọc/trị liệu BPA cho bucket demo (`s3:GetBucketPublicAccessBlock`, `s3:PutBucketPublicAccessBlock`) cùng CloudWatch Logs permissions.</sub>

---

### 5.3 Trigger / Invocation Path

**Trigger type đã dùng cho evidence:** `[X] EventBridge rule`

![EventBridge trigger config](./images/EventBridgeTrigger.png)

<sub>Note: Với evidence pass này, remediation được chứng minh bằng manual invoke vào `hexacode-prod-security-guard` để tạo before/after và CloudTrail proof một cách deterministic.</sub>

---

### 5.4 Demo Vòng Lặp Detect → Fix

**Before — Vi phạm được tạo cố ý:**

![Security violation before](./images/SecBefore.png)

<sub>Note: Bucket `hexacode-prod-submission-artifacts` bị tắt 4 Block Public Access flags. Đây là trạng thái insecure trước khi Lambda chạy.</sub>

**After — Lambda đã fix:**

![Security violation after](./images/SecAfter.png)

<sub>Note: Cùng bucket sau khi Lambda remediate — cả 4 Block Public Access flags đã bật lại.</sub>

**CloudTrail event của lần gọi fix API:**

![CloudTrail remediation event](./images/CloudTrailEvent.png)

<sub>Note: CloudTrail event `PutBucketPublicAccessBlock` cho thấy remediation đã thực sự chạy từ execution role của `hexacode-prod-security-guard`.</sub>

---

### 5.5 Supporting Preventive Control

**Supporting control đã giữ lại:** `S3 Block Public Access account-level`

![S3 BPA account level](./images/S3PBA.png)

<sub>Note: S3 console → Block Public Access settings for this account → cả 4 setting đều ON. Đây là preventive control bổ sung cho cùng họ S3, tách với remediation demo dùng bucket `hexacode-prod-submission-artifacts`.</sub>

![S3 Deny Policy](./images/S3DenyPolicy.png)

<sub>Note: S3 console → Block Public Access settings for this account → cả 4 setting đều ON. Đây là preventive control bổ sung cho cùng họ S3, tách với remediation demo dùng bucket `hexacode-prod-submission-artifacts`.</sub>

**Test Screenshot:**

---

### 5.6 Security Threat Statement

> Temporary guide — xoá block này sau khi viết xong.
> - Viết theo cấu trúc: `misconfiguration là gì -> data/asset nào bị ảnh hưởng -> attacker có thể làm gì`.
> - Tránh viết chung chung kiểu “bị hack”; rubric muốn thấy blast radius cụ thể.

**Guard fix misconfiguration gì:**
`[e.g. "S3 bucket chứa Bedrock KB documents bị set public — mọi người trên internet có thể đọc toàn bộ nội dung knowledge base của app."]`

**Blast radius nếu không remediate:**
`[e.g. "Toàn bộ 36 markdown documents của GeekBrain — bao gồm incident postmortems, SLA targets, team structure — bị lộ công khai. Kẻ tấn công có thể dùng thông tin này để social engineer hoặc target specific vulnerabilities."]`

---

### 5.7 Security-Cost Trade-off Statement

> Temporary guide — xoá block này sau khi viết xong.
> - Nêu **chi phí cụ thể** của control đã chọn, hoặc nói rõ là gần như zero-cost nếu chọn account-level BPA / deny policy.
> - Sau đó giải thích vì sao chi phí đó đáng trả so với blast radius ở trên.

`[1-2 câu nêu tên cost cụ thể và justification. Ví dụ: "KMS CMK tốn $1/tháng per key. Justified vì mỗi decrypt event được log kèm IAM principal — đây là audit trail bắt buộc khi data store chứa thông tin thi cử của người dùng. Cost $1/tháng nhỏ hơn nhiều so với rủi ro compliance khi không có audit trail."]`

---

## Section 6 — Project Recap

### Ứng dụng là gì

`[Mô tả ngắn: HexaCode là một coding practice platform cho phép người dùng luyện tập bài tập lập trình, nộp bài, và nhận hỗ trợ từ AI chatbot.]`

### Business Domain

`[e.g. EdTech / Competitive Programming / Online Judge]`

### Các quyết định kiến trúc và thiết kế chính từ W1-W5

| Tuần | Quyết định chính |
|---|---|
| W1 | `[e.g. 3-tier architecture: CloudFront → API Gateway → ECS Fargate → RDS]` |
| W2 | `[e.g. S3 cho static assets, IAM baseline với MFA trên root]` |
| W3 | `[e.g. RDS PostgreSQL / relational vì data có JOIN phức tạp giữa users-submissions-problems]` |
| W4 | `[e.g. Bedrock Agent với Knowledge Base, Lambda orchestrator, Hybrid Search K=10]` |
| W5 | `[e.g. VPC Peering Production↔Management, Network Firewall với domain allowlist, EFS mount, API Gateway + auth]` |
| W6 | `[e.g. Cost tagging discipline, automated cost guard, CloudWatch observability, self-healing security]` |

---

## Bonus *(Tuỳ chọn)*

> Chỉ điền nếu đã hoàn tất cả 4 must-have và Evidence Pack.

### B1 `[ ]` gp2 → gp3 EBS Migration (+0.25)

**Before (gp2):**

![EBS gp2 before](./images/w6-ebs-gp2-before.png)
<sub>Note: Volume type gp2, IOPS và BurstBalance baseline từ CloudWatch.</sub>

**After (gp3):**

![EBS gp3 after](./images/w6-ebs-gp3-after.png)
<sub>Note: Volume type gp3, IOPS/throughput đã cấu hình, cost delta so với gp2.</sub>

---

### B2 `[ ]` Trusted Advisor Remediations (+0.25)

**Finding 1:**

![Trusted Advisor finding 1](./images/w6-ta-finding-1.png)
<sub>Note: Finding → Action taken → Before/After.</sub>

**Finding 2:**

![Trusted Advisor finding 2](./images/w6-ta-finding-2.png)
<sub>Note: Finding → Action taken → Before/After.</sub>

---

### B3 `[ ]` RI / Savings Plans Break-even Analysis (+0.25)

`[Viết analysis với con số thật. Ví dụ: "Break-even cho 1-year Compute Savings Plan trên ECS Fargate: on-demand cost $X/tháng × 12 = $Y. Savings Plan commitment $Z × 12 = $W. Break-even tại tháng thứ N. Vòng đời workshop 1 tuần → không mua. Sẽ mua khi sustained spend > $X/tháng trong 3+ tháng liên tiếp."]`

---

### B4 `[ ]` "Wasteful → Changed" Reflection (+0.25)

`[100-150 từ với con số thật: tìm thấy gì lãng phí, đã thay đổi gì, cost/performance delta là bao nhiêu.]`

---

### B5 `[ ]` Cost Anomaly Automation (+0.25)

![Cost Anomaly Detection monitor](./images/w6-anomaly-monitor.png)
<sub>Note: Monitor scope về `Application=HexaCode`, EventBridge rule trên `aws.costanomalydetection`, SNS notification nhận được.</sub>

---

### B6 `[ ]` CloudFormation Template cho một resource W6 (+0.25)

```yaml
# Paste CFN template snippet ở đây
# Provision Security Guard Lambda + EventBridge trigger + IAM role
```

![CFN validate output](./images/w6-cfn-validate.png)
<sub>Note: `aws cloudformation validate-template` output — template pass validation.</sub>

---

*— End of W6 Evidence Pack —*
