---
title: "HPC환경에서 Observability 구축하기"
date: 2025-11-14 00:00:00 +/- TTTT
categories: [오픈소스]
tags: [Open Source,kubetail]	# TAG는 반드시 소문자로 이루어져야함!
image: /assets/img/2025-10-21/img0.png
published: false
description: HPC 환경에서 Prometheus와 Grafana 기반의 Observability 파이프라인을 구축하며 배운 점을 정리했다.

---
<style>
  figcaption {
    font-size: 14px;
    color: #555;
    font-style: italic;
  }
</style>



> *HPC 환경에서 Observability를 구축하면서 경험을 정리합니다*

- 시계열 데이터 수집 파이프라인 구축
  수집 하고자 하는 대상
  각 노드에 대한 메트릭 정보
  IB 스위치에 대한 메트릭 정보
  Slurm Job에 대한 메트릭 정보
  GPU 사용에 대한 메트릭 정보
  기본적인 수집 그림 제시

  Slurm Job Exporter 구축에 대한건 따로 글 작성

- 로그 중앙 집중화
  Alloy 이용하여 로그 수집 파이프라인 구축
  relabel 이슈
  카디널리티의 중요성
  Loki 구조와 MinIO 이전의 필요성 (INode 값 확인)

    
- Grafana 대시보드 구축
  시각화를 하는 부분
  어떤 시각화를 통해 어떤 인사이트를 얻을 수 있을지 많은 고민이 필요함
  -> 데이터 분석에 대한 고민이 필요한 부분
  Rule 설정
  AlertManager 구축
  Rule에 대한 한계점 언급
  비정상 탐지
  ML에 대한  
  
-

---

## 🔍 오픈 소스 탐색

