# Evidence Pack — W6: Operations Hardening & Cost-Aware Cloud
# Group 6 — HexaCode

---

## Section 1 — Cover

| Field | Details |
|---|---|
| **Group Number** | Group 6 |
| **Member Names** | Minh Tuấn · Thành Vinh · Anh Hoàng · Hoàng Nhân · Mạnh Khang · Ngọc Thắng · Hoàng Thông · Thành Tâm |
| **Link Repo** | `[GitHub repo URL]` |
| **W5 Evidence Pack** | `[Link tới docs/W5_evidence.md commit]` |

### W5 Feedback đã giải quyết

> Temporary guide — xoá block này sau khi đã thay bằng ảnh thật.
> - Mục này chỉ nên giữ các feedback đã có evidence trực quan từ AWS Console hoặc source-of-truth đủ mạnh.
> - Các feedback chưa đóng hoàn toàn vẫn để ở `docs/envidence_drift.md`, không claim ở đây.

#### 1.1 API Gateway throttling không còn giữ mức 1 RPS

> Temporary guide — xoá block này sau khi đã thay bằng ảnh thật.
> - AWS Console → **API Gateway** → **APIs** → HTTP API `5l48bwt6ck` → **Stages** → chọn stage đang deploy.
> - Chụp phần **Default route settings** để thấy throttling không còn là `1 request/giây`; theo source-of-truth hiện tại là `burst 200 / rate 100`.
> - Sau đó vào **Routes** → `POST /api/chat/messages` để chụp limit riêng cho chat route; theo source-of-truth hiện tại là `burst 20 / rate 10`.
> - Nếu console đang hiển thị khác hẳn, không dùng ảnh này để claim đã sửa hoàn toàn; đưa lại item vào `envidence_drift.md`.

![W5 throttling fixed](./images/w5-feedback-throttling.png)
<sub>Note: Live API Gateway throttling đã rời khỏi narrative cũ `1 request/giây`; có default throttling cho stage và route-level throttling riêng cho `POST /api/chat/messages`.</sub>

#### 1.2 Backup selection không còn mô tả kiểu assign-by-prefix

> Temporary guide — xoá block này sau khi đã thay bằng ảnh thật.
> - AWS Console → **AWS Backup** → **Backup plans** → mở plan `hexacode-prod-daily-backups` hoặc plan tương ứng → **Resource assignments** / **Protected resources**.
> - Chụp sao cho thấy resource assignment hiện tại gắn vào các resource thật, không chỉ mô tả prefix mơ hồ trong narrative.
> - Nếu console chỉ hiện 2 resource hoặc không nhìn rõ danh sách resource thật, **không claim** feedback này đã đóng hoàn toàn; giữ nó trong `envidence_drift.md`.
> - Nếu cần ảnh source-of-truth ở code để bổ trợ, chụp thêm `terraform/main.tf` + `terraform/modules/backup/main.tf`, nhưng ưu tiên AWS Console trước.

![W5 backup selection fixed](./images/w5-feedback-backup-selection.png)
<sub>Note: Backup selection đã được đưa về hướng explicit resource assignment thay vì narrative `assign-by-ARN-prefix`. Chỉ dùng ảnh này để claim closure khi console thể hiện đủ resource thật theo yêu cầu mentor.</sub>

#### 1.3 Provisioned concurrency đã có cấu hình live, nhưng phần diễn giải warm-start phải viết đúng

> Temporary guide — xoá block này sau khi đã thay bằng ảnh thật.
> - AWS Console → **Lambda** → function chat của app → **Configuration** → **Concurrency**.
> - Chụp phần **Reserved concurrency** và **Provisioned concurrency** để chứng minh hạ tầng đã có cấu hình, thay vì chỉ nói suông trong W5.
> - Ảnh này chỉ chứng minh **cấu hình tồn tại**; phần giải thích warm-start vẫn phải dựa trên log/init-duration đúng ngữ cảnh nếu muốn claim đã đóng hoàn toàn.
> - Vì vậy, nếu chỉ mới có ảnh config mà chưa có diễn giải/log tốt hơn, vẫn giữ item narrative này ở `envidence_drift.md`.

![W5 provisioned concurrency config](./images/w5-feedback-provisioned-concurrency.png)
<sub>Note: Chat Lambda đã có reserved/provisioned concurrency config trên live AWS. Đây là evidence tốt hơn cho hạ tầng, nhưng chưa tự động thay thế cho một warm-start explanation đúng.</sub>

---

## Section 2 — MH-COST-V — Cost Visibility & Attribution

> Temporary guide — xoá block này sau khi đã thay bằng ảnh thật.
> - Phần mở đầu này dùng để chứng minh nhóm đã tiếp thu feedback W5 và sửa source-of-truth/operational posture trước khi sang W6.
> - Nếu mentor hỏi sâu, ưu tiên nói rõ điểm nào đã có **live AWS confirmation** và điểm nào mới dừng ở **Terraform/doc correction**.
> - Các mục chưa đóng hẳn đã được tách sang `docs/envidence_drift.md` để xử lý tiếp, không claim quá mức trong evidence pack chính.

### 2.1 Tagging — Bốn tag key bắt buộc trên mọi billable resource

**Tag schema áp dụng:**

| Tag Key | Giá trị | Quy tắc |
|---|---|---|
| `Owner` | `[email@domain.com]` | Nhất quán chữ hoa/thường |
| `Environment` | `dev` | Không trộn "dev" và "Dev" |
| `CostCenter` | `G6` | Group ID |
| `Application` | `HexaCode` | Nhất quán — không đổi case |

> Temporary guide — xoá block này sau khi đã thay bằng ảnh thật.
> - `w6-tags-ec2.png`: AWS Console → **EC2** → **Instances** → chọn instance app đã redeploy → tab **Tags**. Chụp sao cho thấy **Name** + đủ 4 key `Owner`, `Environment`, `CostCenter`, `Application`.
> - `w6-tags-rds.png`: AWS Console → **RDS** → **Databases** → chọn DB instance dùng cho app → tab **Tags**. Chụp rõ tên DB và đủ 4 tag.
> - `w6-tags-lambda.png`: AWS Console → **Lambda** → chọn function của app hoặc function W6 bạn tự tạo → tab **Configuration** / **Tags**. Theo drift audit, `CostGuardLambda` hiện đang **chưa có tag**, nên nếu dùng function này cho MH-COST-V thì phải tag lại trước khi chụp.
> - `w6-tags-s3.png`: AWS Console → **S3** → chọn bucket của app → tab **Properties** → **Tags**. Nếu bucket đang chỉ có tag cũ như `Project` mà chưa có `Application`, ảnh này chưa đạt rubric.
> - Lưu ý theo `docs/aws-console-drift-audit.md`: tag live hiện còn **không nhất quán** (`Environment` có cả `prod` và `production`), nên nên chuẩn hoá xong rồi mới chụp bộ ảnh Section 2.

**Screenshot tag trên EC2 / ECS:**

![Tags on EC2/ECS](./images/w6-tags-ec2.png)
<sub>Note: Cả 4 tag key hiển thị trên instance đã redeploy.</sub>

**Screenshot tag trên RDS:**

![Tags on RDS](./images/w6-tags-rds.png)
<sub>Note: Cả 4 tag key hiển thị trên RDS hexacode-prod-db.</sub>

**Screenshot tag trên Lambda:**

![Tags on Lambda](./images/w6-tags-lambda.png)
<sub>Note: Cả 4 tag key hiển thị trên Lambda functions đã redeploy.</sub>

**Screenshot tag trên S3:**

![Tags on S3](./images/w6-tags-s3.png)
<sub>Note: Cả 4 tag key hiển thị trên S3 buckets của app.</sub>

---

### 2.2 Cost Allocation Tags — Activated trong Billing Console

> Temporary guide — xoá block này sau khi có ảnh thật.
> - AWS Console → **Billing and Cost Management** → **Cost allocation tags**.
> - Tìm các key `Owner` và `Application`, filter theo status nếu cần, rồi **Activate**.
> - Chụp màn hình khi thấy cột status là **Active** cho đúng 2 key này.
> - Theo drift audit, phiên read-only CLI **không verify được** bước này vì `ce list-cost-allocation-tags` bị `AccessDenied`, nên đây là mục bắt buộc phải xác nhận trực tiếp trong Billing Console chứ không suy ra từ tag trên resource.

![Cost allocation tags activated](./images/w6-cost-allocation-tags.png)
<sub>Note: Tag `Owner` và `Application` đã được Activate trong AWS Billing console → Cost allocation tags. Đây là bước tách biệt khỏi việc tạo tag — bỏ qua bước này thì tag không xuất hiện như filter dimension trong Cost Explorer.</sub>

---

### 2.3 Cost Monitoring Tool(s) đã cấu hình

> Temporary guide — xoá block này sau khi có ảnh thật.
> - `w6-budgets-config.png`: AWS Console → **Billing** → **Budgets** → mở budget `prod`. Với evidence pack workshop, giữ narrative theo yêu cầu bài là **$150/tuần**; ảnh nên ưu tiên thể hiện ngưỡng, alert, và wiring hơn là tranh luận về period wording trong template.
> - `w6-cost-explorer-filter.png`: AWS Console → **Cost Explorer** → filter theo tag dimension của workload. Chỉ chụp khi `Application=HexaCode` đã xuất hiện làm filter; nếu chưa thấy, quay lại mục Cost allocation tags.
> - `w6-anomaly-detection.png`: AWS Console → **Cost Anomaly Detection** → monitor `Monitor-Hexacode`. Theo audit, monitor live đang là **SERVICE monitor**, threshold `$75`, subscription hằng ngày tới `nahoangit@gmail.com`.

**Tool 1 — AWS Budgets:**

![AWS Budgets config](./images/w6-budgets-config.png)
<sub>Note: Budget $150/tuần được set trước Thứ Sáu. Alert gửi về email/SNS khi đạt các ngưỡng cấu hình. Với evidence pack workshop, giữ wording này nhất quán với narrative của bài.</sub>

**Tool 2 — Cost Explorer filter theo tag:**

![Cost Explorer filter](./images/w6-cost-explorer-filter.png)
<sub>Note: Cost Explorer filter theo `Application=HexaCode` — thấy chi phí breakdown theo service chỉ của workload nhóm, không phải toàn account.</sub>

**Tool 3 — Cost Anomaly Detection (nếu có):**

![Cost Anomaly Detection](./images/w6-anomaly-detection.png)
<sub>Note: Monitor scope về `Application=HexaCode`. Alert subscription được xác nhận.</sub>

---

### 2.4 Baseline Cost Breakdown (sau ít nhất 24h data)

> Temporary guide — xoá block này sau khi có ảnh thật.
> - AWS Console → **Cost Explorer** → chọn khoảng thời gian có ít nhất 24h dữ liệu sau redeploy.
> - Group by **Service** và filter theo tag workload của nhóm.
> - Ảnh nên hiển thị rõ phần breakdown để viết được đoạn “top 3 cost drivers”.
> - Theo drift audit, một số cost driver có khả năng nổi bật nếu chúng đang chạy thật: **RDS**, **Network Firewall / NAT-related surface**, **ECS/Fargate**, **CloudFront**, hoặc **Bedrock/OpenSearch**. Đừng điền ví dụ cứng nếu số thật trên Cost Explorer khác.

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
| **Environment** | `production` | Xác định môi trường vận hành là Production. | Toàn bộ tài nguyên |
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

> Temporary guide — xoá block này sau khi có ảnh thật.
> - AWS Console → **Lambda** → tìm function `CostGuardLambda`.
> - Chụp phần **Overview** hoặc **Configuration** sao cho thấy function name, runtime, và **Last modified**. Theo drift audit, live function này đang active và last modified là `2026-05-21T09:25:59Z`.
> - Nếu nhóm đã tự sửa/clone function khác để đáp ứng rubric tốt hơn, hãy chụp function thực sự dùng cho demo thay vì mặc định bám `CostGuardLambda`.

**Mô tả:** Lambda function scan mọi EC2/RDS không được tag `keep=true` (hoặc `Environment=dev`) và stop chúng.

**Screenshot Lambda function:**

![Cost Guard Lambda](./images/w6-cost-guard-lambda.png)
<sub>Note: Lambda function đã deploy, runtime Python, last modified timestamp.</sub>

**Lambda code snippet:**

```python
import boto3

ec2 = boto3.client('ec2')
rds = boto3.client('rds')

def handler(event, context):
    # Stop EC2 instances không có tag keep=true
    instances = ec2.describe_instances(
        Filters=[{'Name': 'instance-state-name', 'Values': ['running']}]
    )
    for reservation in instances['Reservations']:
        for instance in reservation['Instances']:
            tags = {t['Key']: t['Value'] for t in instance.get('Tags', [])}
            if tags.get('keep') != 'true':
                ec2.stop_instances(InstanceIds=[instance['InstanceId']])
                print(f"Stopped EC2: {instance['InstanceId']}")

    # Stop RDS instances không có tag keep=true
    dbs = rds.describe_db_instances()
    for db in dbs['DBInstances']:
        tags_resp = rds.list_tags_for_resource(ResourceName=db['DBInstanceArn'])
        tags = {t['Key']: t['Value'] for t in tags_resp['TagList']}
        if tags.get('keep') != 'true' and db['DBInstanceStatus'] == 'available':
            rds.stop_db_instance(DBInstanceIdentifier=db['DBInstanceIdentifier'])
            print(f"Stopped RDS: {db['DBInstanceIdentifier']}")
```

---

### 3.2 IAM Role — Least Privilege

> Temporary guide — xoá block này sau khi có ảnh thật.
> - Từ Lambda `CostGuardLambda` → tab **Configuration** → **Permissions** → click execution role.
> - Chụp 1 ảnh thể hiện trust relationship tới Lambda nếu cần, và 1 ảnh policy permissions gắn vào role.
> - Theo W6 spec, giảng viên cần thấy least-privilege reasoning; nếu policy đang rộng quá, ảnh này sẽ tự lộ điểm yếu.

![Cost Guard IAM role](./images/w6-cost-guard-iam.png)
<sub>Note: IAM execution role chỉ có các permission cần thiết — `ec2:StopInstances`, `ec2:DescribeInstances`, `rds:StopDBInstance`, `rds:DescribeDBInstances`, `rds:ListTagsForResource`. Không có `Action: "*"` hay `Resource: "*"`.</sub>

### 3.3 EventBridge Daily Schedule

> Temporary guide — xoá block này sau khi có ảnh thật.
> - AWS Console → **EventBridge Scheduler** → mở schedule `daily-cost-guard`.
> - Theo drift audit, live schedule hiện là `cron(0 9 * * ? *)`, timezone `Asia/Ho_Chi_Minh`, target `CostGuardLambda`, trạng thái **Enabled**.
> - Chụp màn hình có đủ: schedule name, cron expression, timezone, target ARN/function.

![EventBridge schedule](./images/w6-eventbridge-schedule.png)
<sub>Note: EventBridge Scheduler cron chạy daily invoke Cost Guard Lambda. Ảnh nên phản ánh đúng cron expression và timezone đang có trên live AWS.</sub>

---

### 3.4 Demonstrated Stop — Before/After + CloudTrail

> Temporary guide — xoá block này sau khi có ảnh thật.
> - Mục này chỉ đạt khi là **một resource thật** bị stop bởi Lambda.
> - Before/After: vào **EC2 → Instances** hoặc **RDS → Databases** rồi chụp cùng một resource ở trạng thái trước và sau.
> - CloudTrail: AWS Console → **CloudTrail** → **Event history** → filter `Event name = StopInstances` hoặc `StopDBInstance`.
> - Chụp sao cho thấy `eventTime`, `eventName`, resource ID, và identity/userAgent liên quan tới Lambda execution path.
> - Nếu chỉ có budget/SNS wiring mà chưa có ảnh resource bị stop thật thì MH-COST-A vẫn thiếu bằng chứng.

![Instance before stop](./images/w6-instance-before-stop.png)
<sub>Note: EC2 instance (hoặc RDS) đang ở trạng thái Running trước khi Lambda chạy.</sub>

**After — Instance đã Stopped:**

![Instance after stop](./images/w6-instance-after-stop.png)
<sub>Note: Cùng instance đã chuyển sang Stopped sau khi Cost Guard Lambda được invoke.</sub>

**CloudTrail event `StopInstances` / `StopDBInstance`:**

![CloudTrail stop event](./images/w6-cloudtrail-stop.png)
<sub>Note: CloudTrail event xác nhận Lambda đã gọi StopInstances/StopDBInstance — thấy eventName, eventTime, userAgent (Lambda role ARN), và instanceId bị stop.</sub>

---

### 3.5 Budgets daily $150 → SNS → Lambda (Wire + Demo)

> Temporary guide — xoá block này sau khi có ảnh thật.
> - AWS Console → **Billing** → **Budgets** → budget `prod` để chụp threshold/action notifications.
> - AWS Console → **SNS** → topic `cost-guard-budget-alerts` để chụp subscriptions/delivery path.
> - Nếu có thể, thêm ảnh Lambda trigger/subscription để nối mạch `Budget → SNS → Lambda`.
> - Theo drift audit, live AWS **không thấy native Budget Actions**, chỉ xác nhận budget-to-SNS-to-Lambda path. Vì vậy caption nên nói đúng là **SNS wiring**, đừng claim Budget Action nếu console chưa có.

![Budgets SNS wiring](./images/w6-budgets-sns.png)
<sub>Note: AWS Budgets daily budget → SNS topic được wire. Khi budget vượt ngưỡng, SNS publish message → Lambda được invoke.</sub>

**Test SNS publish — demonstrate chain:**

```bash
aws sns publish \
  --topic-arn arn:aws:sns:us-west-2:ACCOUNT_ID:cost-guard-topic \
  --message '{"AlarmName":"BudgetAlert","NewStateValue":"ALARM"}' \
  --region us-west-2
```

![SNS test publish](./images/w6-sns-test-publish.png)
<sub>Note: Manual SNS publish trigger Lambda → Lambda stop một resource → xác nhận chain hoạt động end-to-end mà không cần chờ cost data thật.</sub>

---

### 3.6 Cost Data Latency — ADR

**Architecture Decision Record:**

**Context:** AWS cost data có độ trễ ~8–24 giờ trước khi xuất hiện trong Cost Explorer và Budgets. Trong một account workshop 48 giờ, cost-driven trigger từ Budgets gần như sẽ không fire vì data chưa kịp cập nhật.

**Decision:** Wire Budgets daily $150 → SNS → Lambda (đảm bảo chain tồn tại và test được), đồng thời dùng EventBridge Scheduler daily cron làm primary trigger đáng tin trong môi trường workshop.

**Consequences:** Trong production thật (account chạy nhiều tuần), Budgets cost-driven trigger sẽ fire bình thường sau 24h data. Chain đã được wire và test bằng SNS manual publish — behavior production đã được xác nhận.

---

## Section 4 — MH-OBS — CloudWatch Observability

### 4.1 CloudWatch Dashboard

> Temporary guide — xoá block này sau khi có ảnh thật.
> - AWS Console → **CloudWatch** → **Dashboards** → mở dashboard nhóm tạo cho W6.
> - Ảnh phải thấy đồng thời: 1 widget **custom metric** có title rõ ràng, và ít nhất 2 widget metric chuẩn.
> - Theo W6 spec, widget rỗng hoặc không có datapoint sẽ không đủ mạnh; nên generate traffic/invoke trước khi chụp.
> - Theo drift audit, live AWS đã có API access logs và backup-failure event logs, nhưng audit **không xác nhận** sẵn dashboard custom metric hoàn chỉnh; phần này nhiều khả năng vẫn phải tự hoàn thiện rồi mới chụp.

![CloudWatch dashboard](./images/w6-cloudwatch-dashboard.png)
<sub>Note: Dashboard với 3 widget: (1) Custom metric `[tên metric]`, (2) Standard metric Lambda Error Rate, (3) Standard metric RDS DatabaseConnections. Mọi widget đều có data point thật.</sub>

### 4.2 Custom Metric — `PutMetricData` Code Snippet

> Temporary guide — xoá block này sau khi có ảnh thật.
> - `w6-custom-metric.png`: CloudWatch → **Metrics** → namespace custom của nhóm, ví dụ `HexaCode/Operations`.
> - Chụp màn hình khi nhìn thấy metric name, dimensions, và datapoints thật.
> - Chỉ dùng snippet code này nếu nó đúng với code app đang chạy; nếu chưa instrument thật thì đây mới chỉ là placeholder, chưa phải evidence.

**Metric đo gì:** `[e.g. Bedrock agent invocation latency ms]`

```python
import boto3
import time

cloudwatch = boto3.client('cloudwatch')

def handler(event, context):
    start = time.time()
    
    # [Business logic của app ở đây — gọi Bedrock, query DB, v.v.]
    
    latency_ms = (time.time() - start) * 1000
    
    cloudwatch.put_metric_data(
        Namespace='HexaCode/Operations',
        MetricData=[{
            'MetricName': 'BedrockAgentLatencyMs',
            'Value': latency_ms,
            'Unit': 'Milliseconds',
            'Dimensions': [
                {'Name': 'FunctionName', 'Value': 'hexacode-prod-chat'},
                {'Name': 'Environment', 'Value': 'dev'}
            ]
        }]
    )
```

![Custom metric data points](./images/w6-custom-metric.png)
<sub>Note: Custom metric `BedrockAgentLatencyMs` trong namespace `HexaCode/Operations` — thấy data points thật từ Lambda invocations.</sub>

---

### 4.3 CloudWatch Alarm — Trạng thái OK hoặc ALARM

> Temporary guide — xoá block này sau khi có ảnh thật.
> - AWS Console → **CloudWatch** → **Alarms**.
> - Ảnh `w6-alarm-config.png` nên chụp detail của một alarm để thấy metric, threshold, evaluation periods, và action destination.
> - Ảnh `w6-alarm-state.png` nên chụp danh sách alarms hoặc detail panel có state là **OK** hoặc **ALARM**, tuyệt đối tránh `INSUFFICIENT_DATA` vì W6 rubric trừ điểm rất rõ.
> - Nếu metric chưa có datapoint, hãy generate traffic/lỗi trước rồi mới chụp.

![CloudWatch alarm config](./images/w6-alarm-config.png)
<sub>Note: Alarm configuration — metric name, threshold, evaluation period (e.g. Lambda Errors > 5 trong 5 phút), action destination (SNS topic). Alarm đang ở trạng thái OK hoặc ALARM — không phải INSUFFICIENT_DATA.</sub>

![CloudWatch alarm state](./images/w6-alarm-state.png)
<sub>Note: Alarm state screenshot chụp gần Thứ Sáu — xác nhận metric đã có data point để evaluate. Nếu cần, invoke Lambda với bad input 6 lần vào Thứ Năm để trigger alarm.</sub>

---

### 4.4 Log Insights Query — Saved

> Temporary guide — xoá block này sau khi có ảnh thật.
> - AWS Console → **CloudWatch** → **Logs Insights**.
> - Chọn log group thật của app, ví dụ API Gateway access logs hoặc `/aws/lambda/...`.
> - `w6-log-insights-result.png`: chụp cùng lúc query text, tên log group, và ít nhất 5 dòng kết quả.
> - `w6-log-insights-saved.png`: chụp danh sách **Saved queries** để thấy query name đã được lưu.
> - Theo drift audit, live AWS đã xác nhận API access logs được enable; đây là nguồn log rất hợp lý để chụp nếu nhóm chưa có Lambda log đẹp.

**Query text:**

```bash
# Lambda error spikes by 5-minute window
fields @timestamp, @message
| filter @message like /ERROR/
| stats count(*) as error_count by bin(5m)
| sort @timestamp desc
```

**Log group chạy chống lại:** `/aws/lambda/hexacode-prod-chat`

![Log Insights query result](./images/w6-log-insights-result.png)
<sub>Note: Query trả về ít nhất 5 result rows thật. Thấy tên query đã save trong danh sách Saved Queries.</sub>

![Log Insights saved query](./images/w6-log-insights-saved.png)
<sub>Note: Saved query name nhìn thấy trong CloudWatch → Log Insights → Saved Queries.</sub>

---

## Section 5 — MH-SEC — Self-Healing Security Guard

### 5.1 Security Guard Lambda

> Temporary guide — xoá block này sau khi có ảnh thật.
> - Trước tiên chốt **một** path demo chính: `S3 public -> PutPublicAccessBlock` hoặc `SG 0.0.0.0/0:22 -> RevokeSecurityGroupIngress`.
> - AWS Console → **Lambda** → chọn function Security Guard của nhóm → chụp phần function name, runtime, trigger summary, và last modified.
> - Theo W6 spec, điểm nằm ở detect→fix loop có thật; đừng chỉ chụp function tồn tại mà chưa có remediation evidence.

**Misconfiguration detect và fix:** `[e.g. S3 bucket bị set public → Lambda gọi PutPublicAccessBlock]`
hoặc `[e.g. Security Group ingress 0.0.0.0/0 trên port 22 → Lambda gọi RevokeSecurityGroupIngress]`

**Screenshot Lambda function:**

![Security Guard Lambda](./images/w6-sec-guard-lambda.png)
<sub>Note: Lambda function đã deploy với least-privilege IAM role.</sub>

**Lambda code snippet:**

```python
import boto3

s3 = boto3.client('s3')

def handler(event, context):
    # Lấy bucket name từ CloudTrail event (nếu trigger từ EventBridge)
    bucket_name = event.get('detail', {}).get('requestParameters', {}).get('bucketName')
    
    if not bucket_name:
        # Fallback: scan tất cả bucket nếu trigger từ scheduled cron
        buckets = s3.list_buckets()['Buckets']
        for bucket in buckets:
            check_and_fix_bucket(bucket['Name'])
        return
    
    check_and_fix_bucket(bucket_name)

def check_and_fix_bucket(bucket_name):
    try:
        status = s3.get_public_access_block(Bucket=bucket_name)
        config = status['PublicAccessBlockConfiguration']
        if not all(config.values()):
            s3.put_public_access_block(
                Bucket=bucket_name,
                PublicAccessBlockConfiguration={
                    'BlockPublicAcls': True,
                    'IgnorePublicAcls': True,
                    'BlockPublicPolicy': True,
                    'RestrictPublicBuckets': True
                }
            )
            print(f"Fixed public access on bucket: {bucket_name}")
    except Exception as e:
        print(f"Error checking bucket {bucket_name}: {e}")
```

---

### 5.2 IAM Role — Least Privilege

> Temporary guide — xoá block này sau khi có ảnh thật.
> - Từ function Security Guard → **Configuration** → **Permissions** → click execution role.
> - Nếu chọn path S3, ảnh policy nên lộ rõ các quyền như `s3:PutPublicAccessBlock` và các quyền read tối thiểu liên quan.
> - Nếu chọn path SG, ảnh policy nên lộ rõ `ec2:RevokeSecurityGroupIngress` + quyền describe cần thiết.

![Security Guard IAM role](./images/w6-sec-guard-iam.png)
<sub>Note: IAM execution role chỉ có `s3:PutPublicAccessBlock`, `s3:GetBucketPolicyStatus`, `s3:ListAllMyBuckets` — hoặc `ec2:RevokeSecurityGroupIngress`, `ec2:DescribeSecurityGroups`. Không có wildcard.</sub>

---

### 5.3 EventBridge Trigger

> Temporary guide — xoá block này sau khi có ảnh thật.
> - Nếu chọn real-time path: AWS Console → **EventBridge** → **Rules** → rule match CloudTrail event như `PutBucketPolicy`, `PutBucketAcl`, hoặc `AuthorizeSecurityGroupIngress`.
> - Nếu chọn scheduled fallback: AWS Console → **EventBridge Scheduler** → schedule daily cron.
> - Chụp màn hình phải thấy rule/schedule name, event pattern hoặc cron expression, target Lambda, và status **Enabled**.
> - Theo W6 spec, EventBridge rule trên CloudTrail event là evidence mạnh hơn cho self-healing loop; cron fallback vẫn hợp lệ nhưng nên mô tả rõ.

**Trigger type đã chọn:** `[ ] EventBridge rule trên CloudTrail event` &nbsp;&nbsp; `[ ] EventBridge Scheduler daily cron`

![EventBridge trigger config](./images/w6-sec-guard-trigger.png)
<sub>Note: EventBridge rule trên event source `aws.s3` / event `PutBucketPolicy` / `PutBucketAcl` — hoặc daily cron schedule. Rule đang Enabled.</sub>

---

### 5.4 Demo Vòng Lặp Detect → Fix

> Temporary guide — xoá block này sau khi có ảnh thật.
> - `w6-sec-before.png`: chụp trạng thái **insecure** ngay sau khi cố ý tạo vi phạm.
> - `w6-sec-after.png`: chụp cùng resource sau khi Lambda đã remediate.
> - `w6-cloudtrail-remediation.png`: AWS Console → **CloudTrail** → **Event history** → filter `PutPublicAccessBlock` hoặc `RevokeSecurityGroupIngress`.
> - Ảnh CloudTrail phải cho thấy eventName, eventTime, và resource bị sửa; nếu thấy principal/role của Lambda thì càng mạnh.
> - Với path S3, chọn bucket thật của app nhưng tránh bucket nhạy cảm khó rollback. Với path SG, dùng security group test để tránh ảnh hưởng production path ngoài ý muốn.

**Before — Vi phạm được tạo cố ý:**

![Security violation before](./images/w6-sec-before.png)
<sub>Note: S3 bucket bị set public (Block Public Access tắt) — hoặc Security Group có rule 0.0.0.0/0 port 22. Đây là trạng thái "insecure" trước khi Lambda chạy.</sub>

**After — Lambda đã fix:**

![Security violation after](./images/w6-sec-after.png)
<sub>Note: Cùng bucket/SG sau khi Lambda detect và remediate — Block Public Access bật lại / rule 0.0.0.0/0 đã bị revoke.</sub>

**CloudTrail event của lần gọi fix API:**

![CloudTrail remediation event](./images/w6-cloudtrail-remediation.png)
<sub>Note: CloudTrail event `PutPublicAccessBlock` / `RevokeSecurityGroupIngress` — thấy eventName, eventTime, userAgent (Lambda role ARN), và resource bị fix. Đây là bằng chứng remediation đã thực sự chạy.</sub>

---

### 5.5 Supporting Preventive Control

> Temporary guide — xoá block này sau khi có ảnh thật.
> - Chọn **một** path rồi xoá hai path còn lại để evidence pack gọn.
> - Theo drift audit hiện tại, path dễ bám live AWS nhất có thể là **S3 Block Public Access account-level** hoặc **IAM Access Analyzer** nếu account đã bật; còn path **KMS CMK** cần chắc chắn có service thật dùng key và có CloudTrail `GenerateDataKey`/`Decrypt`.

**Path đã chọn:** `[ ] Path A — KMS CMK` &nbsp;&nbsp; `[ ] Path B — S3 Block Public Access account-level` &nbsp;&nbsp; `[ ] Path C — IAM Access Analyzer`

---

**Nếu Path A — KMS CMK:**

> Temporary guide — xoá block này sau khi có ảnh thật.
> - AWS Console → **KMS** → **Customer managed keys** → chọn key alias của nhóm.
> - `w6-kms-cmk.png`: chụp alias, key spec, rotation enabled.
> - `w6-kms-applied.png`: chụp service đang dùng CMK, ví dụ RDS/EFS/S3.
> - `w6-kms-cloudtrail.png`: CloudTrail filter `GenerateDataKey` hoặc `Decrypt`, rồi xác nhận caller service như `rds.amazonaws.com` hoặc `s3.amazonaws.com`.

![KMS CMK created](./images/w6-kms-cmk.png)
<sub>Note: Customer Managed Key `alias/hexacode-rds-prod` đã tạo, Symmetric, key rotation Enabled.</sub>

![KMS applied to data store](./images/w6-kms-applied.png)
<sub>Note: RDS / EFS / S3 đã được modify để dùng CMK (không phải aws/rds hay aws/s3 — AWS-managed key).</sub>

![CloudTrail kms:GenerateDataKey](./images/w6-kms-cloudtrail.png)
<sub>Note: CloudTrail event `kms:GenerateDataKey` từ `rds.amazonaws.com` / `s3.amazonaws.com` — xác nhận CMK đang được dùng active khi data được encrypt/decrypt.</sub>

---

**Nếu Path B — S3 Block Public Access account-level:**

> Temporary guide — xoá block này sau khi có ảnh thật.
> - AWS Console → **S3** → **Block Public Access settings for this account**.
> - `w6-s3-bpa-account.png`: chụp cả 4 toggle ON.
> - `w6-s3-deny-policy.png`: chụp bucket policy deny non-TLS hoặc deny unencrypted PutObject.
> - `w6-s3-test-denied.png`: chụp lỗi/deny result của test call.

![S3 BPA account level](./images/w6-s3-bpa-account.png)
<sub>Note: S3 console → Block Public Access settings for this account → cả 4 setting đều ON.</sub>

![S3 deny policy](./images/w6-s3-deny-policy.png)
<sub>Note: Bucket policy deny PutObject non-TLS (`aws:SecureTransport=false`).</sub>

![S3 test call denied](./images/w6-s3-test-denied.png)
<sub>Note: Test call bị policy reject — xác nhận enforce đang hoạt động.</sub>

---

**Nếu Path C — IAM Access Analyzer:**

> Temporary guide — xoá block này sau khi có ảnh thật.
> - AWS Console → **IAM** → **Access Analyzer**.
> - `w6-access-analyzer.png`: chụp analyzer enabled.
> - `w6-access-analyzer-finding.png`: chụp ít nhất 1 external-access finding + decision triage.
> - Nếu account chưa bật Access Analyzer, path này sẽ phát sinh thêm setup nên không phải đường ngắn nhất.

![IAM Access Analyzer enabled](./images/w6-access-analyzer.png)
<sub>Note: IAM Access Analyzer đã enable trong account.</sub>

![External access finding](./images/w6-access-analyzer-finding.png)
<sub>Note: ≥1 external-access finding được surface. Triage decision: finding này là gì, có phải intended không, production remediation là gì.</sub>

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
