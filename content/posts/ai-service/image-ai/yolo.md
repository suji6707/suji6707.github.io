+++
title = 'YOLO'
date = 2025-03-26T22:50:44+09:00
draft = true
+++

YOLOv8 ONNX 모델은 .onnx 확장자를 가진 단일 파일로 제공됩니다. 이 파일은 모델 구조와 학습된 weight를 모두 포함합니다.
TensorFlow.js 모델은 model.json 파일 (모델 구조 정의) 과 여러 개의 .bin 파일 (weight 데이터) 로 구성됩니다.

tensorflowjs_converter

Detection은 넓은 범위에서 물체의 위치를 파악
Segmentation은 디테일한 물체의 윤곽을 추출. 배경과의 경계를 정확히 구분해야 하는 경우.

Object Detection을 사용해 어떤 물체가 있는지 찾고, Segmentation을 통해 해당 물체를 픽셀 단위로 정밀하게 처리한 뒤, 원하는 작업(이동, 제거 등)을 수행

AI 라이브러리를 활용해 아주 간단한 이미지 편집기를 만들어줘. 이미지속 특정 물체를 클릭하면 해당 물체를 식별해서 움직이거나 제거할 수 있어야 해. 아래 라이브러리들을 활용할거고, 서비스단은 typescript를 쓰고싶어. 
1. Object Detection: YOLO를 사용해 이미지에서 특정 물체를 식별하고 위치(Bounding Box)를 찾아냅니다.
2. Segment Anything Model: 물체의 경계를 픽셀 단위로 추출합니다.
이미지 편집 라이브러리
3. 이미지 편집: Pillow 또는 OpenCV를 사용해 해당 물체를 이동하거나 제거하며, 필요한 경우 Stable Diffusion inpainting을 활용해 자연스러운 배경 채우기를 구현합니다.


프론트엔드 (http://localhost:1234)
사용자 인터페이스를 제공하고 이미지 편집 기능 수행
api.ts 파일을 통해 백엔드 API와 통신

백엔드 (http://localhost:5000)
API 엔드포인트:
/api/upload: 이미지 업로드
/api/detect: 객체 감지
/api/segment: 객체 분할
/api/move: 객체 이동
/api/remove: 객체 제거
/api/save: 이미지 저장
/api/images/<filename>: 이미지 파일 제공

데이터 흐름:
사용자가 이미지를 업로드하면, 프론트엔드가 백엔드로 전송
백엔드는 AI 모델을 사용하여 이미지 처리
처리 결과가 프론트엔드로 반환되어 사용자에게 표시됨
6. 웹 브라우저에서 접속
웹 브라우저를 열고 http://localhost:1234 에 접속하면 애플리케이션 사용 가능



원인 분석
setState를 호출하면 React는 컴포넌트를 다시 렌더링합니다. 그러나 Canvas 요소는 React의 가상 DOM과 별개로 작동하기 때문에, 상태 업데이트가 발생할 때마다 캔버스 콘텐츠는 초기화됩니다.

특히 이미지를 로드하고 그린 직후에 setState가 호출되면 다음 순서로 진행됩니다:

이미지를 캔버스에 그립니다
setState가 호출되어 컴포넌트가 리렌더링됩니다
리렌더링 과정에서 캔버스 콘텐츠가 초기화됩니다
이미지가 사라집니다
해결 방법
상태 업데이트와 캔버스 작업 분리하기: 객체 상자 크기 조정 상태 업데이트는 이미지 렌더링 이후 별도 useEffect에서 처리합니다.
상태 업데이트 후 이미지 다시 그리기:

* Canvas 이미지 비율 유지하면서 그리기
스크린 오프 방법

🟣🟣🟣🟣 
awesome 에서 repo들 다 있음.
레포 받아서 폴더구조랑 파일 등 계속 물어보면서 나한테 적용해보기.
edit anything 레포 받아서 ㄱㄱ. 
