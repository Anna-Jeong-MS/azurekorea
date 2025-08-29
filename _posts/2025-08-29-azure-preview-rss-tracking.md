---
layout: post
title: "Azure Preview 서비스 변경 사항을 RSS로 추적하기"
author: wonsungso
categories: [Solution]
tags: [Azure, Preview, RSS, Updates, Useful Tips]
---

Azure에서는 새로운 기능이 **Preview** 상태로 먼저 공개된 뒤, 일정 기간 후에 정식 출시(GA, Generally Available)되는 경우가 많습니다.  
Preview 단계에서는 새로운 서비스를 조기 체험할 수 있는 장점이 있지만, 기능이나 API가 변경되거나 사라질 수도 있어 운영 환경에서는 신중하게 접근해야 합니다.  

이러한 이유로 **Preview 기능을 꾸준히 모니터링하는 것**은 매우 중요합니다.  
- **아키텍처 설계 시**: 현재 Preview 단계인 기능을 활용할지, 정식 GA까지 기다릴지 판단  
- **서비스 안정성 확보**: Preview 기능을 실험적으로 적용해보고 GA 전 변경사항에 대비  
- **기술 트렌드 파악**: Microsoft Azure에서 어떤 서비스에 집중하고 있는지 조기 파악 가능

Microsoft는 [Azure Updates](https://azure.microsoft.com/updates)와 [Release Communications RSS](https://www.microsoft.com/releasecommunications/api/v2/azure/rss)를 통해 서비스 변경 사항을 공지합니다.  
이번 글에서는 이 RSS 데이터를 활용하여, **최근 7일 간 공개된 'In Preview' 항목만 자동으로 필터링**해주는 Python 스크립트를 작성하고 실행하는 방법을 소개합니다.  
이 스크립트는 단순하지만, 실제 운영팀에서 **Azure 변경 사항을 빠르게 캐치**하고, 사내 공유용 리포트를 생성하는 데 유용하게 활용할 수 있습니다.

---

## 사전 준비
- Python 3.8 이상
- `feedparser` 패키지 설치
```bash
pip install feedparser
```
---

## 코드 예시
> ⚠️ **Disclaimer**  
> 본 문서는 학습 및 실습을 위한 예제이며, 실제 프로덕션 환경에 적용하기 전 반드시 보안 및 운영 가이드라인을 검토하시기 바랍니다. 작성된 모든 코드와 예시는 _"있는 그대로(as-is)"_ 제공되며, Microsoft 또는 작성자는 이에 대한 책임을 지지 않습니다.

```python
"""
이 코드는 Azure 업데이트 RSS에서 'In Preview' 상태의 항목만 필터링하여 최근 7일간의 내역을 출력하는 예시입니다.
이는 샘플 코드이며, 실제 서비스 환경이나 중요한 워크로드에 적용 시 모든 책임은 사용자에게 있습니다.
"""

import feedparser
from datetime import datetime, timedelta, timezone

# RSS 피드 URL
RSS_URL = "https://www.microsoft.com/releasecommunications/api/v2/azure/rss"

# 날짜 범위 계산
today = datetime.now(timezone.utc).date()
seven_days_ago = today - timedelta(days=7)

# RSS 피드 파싱
feed = feedparser.parse(RSS_URL)

print(f"Azure 업데이트 - '미리 보기(In Preview)' (최근 7일: {seven_days_ago} ~ {today})")
print("=" * 80)

count = 0
for entry in feed.entries:
    title = entry.title
    link = entry.link
    pub_date = datetime(*entry.published_parsed[:6], tzinfo=timezone.utc).date()

    # 제목을 소문자로 치환 후 'in preview' 포함 여부를 확인
    title_lc = title.lower()

    # 최근 7일 이내 + 'in preview' 키워드 포함(대소문자 무관)
    if seven_days_ago <= pub_date <= today and ("in preview" in title_lc):
        count += 1
        print(f"[{pub_date}] {title}")
        print(f"👉 링크: {link}")
        print("-" * 80)

if count == 0:
    print("최근 7일간 '미리 보기(In Preview)' 상태의 새로운 항목이 없습니다.")
```

---

## 실행 예시
스크립트를 실행하면 아래와 같이 최근 7일간의 In Preview 업데이트만 출력됩니다.
![스크립트 실행 결과](../assets/images/wonsungso/2025-08-29-azure-preview-rss-tracking/1_preview_result.png)

---

## 활용 방안
- **매일 실행**: `cron`이나 Windows 작업 스케줄러에 등록하여 매일 아침 자동 실행
- **Slack/Teams 알림 연동**: 출력 결과를 메시지로 변환하여 채팅 채널에 발송
- **데이터 저장**: DB 및 기타 저장소에 저장하여 장기 추이 분석

---

## 마무리

Preview 서비스는 출시 이후에도 기능 변경이나 사양 변경 가능성이 크기 때문에,  
자동화 스크립트를 통해 주기적으로 모니터링하면 서비스 안정성 확보와 변경 대응에 큰 도움이 됩니다.  

특히, 운영 환경에서는 Preview 기능 사용을 최소화해야 하지만, **PoC(Proof of Concept)나 사내 테스트 환경**에서는 적극적으로 활용해보는 것이 좋습니다.  
이 과정에서 RSS 기반 자동화는 다음과 같은 이점을 제공합니다:  
- **변경 사항 사전 인지**: GA 전에 미리 테스트하여 리스크를 줄일 수 있음  
- **업데이트 내역 공유**: 팀 내/조직 내 주간 보고 자료로 바로 활용 가능  
- **확장성**: 간단한 스크립트를 발전시켜 이메일, Teams 등 알림 채널로 확장 가능  

이를 통해 운영팀은 매일 아침 새로운 Preview 기능을 자동으로 받아볼 수 있고, 서비스 안정성 및 클라우드 전략 수립에 한 발 앞서 대응할 수 있습니다.
