https://rclone.org/dropbox/
https://docs.runpod.io/sdks/python/overview

위 도구들을 이용해서, pytorch 도커 이미지를 사용하는 인스턴스를 생성한 후에 자동으로 학습을 시작할 수 있는 스크립트를 만들어줘.

1. 드롭박스의 특정 디렉토리를 /workspace와 싱크 (from dropbox)
2. cd /workspace && pip install -r requirements.txt
3. nohup python scripts/train.py --config configs/acoustic.yaml --exp_name acoustic &
4. 로그 확인
5. 위 작업 완료후 nohup.out의 업데이트가 없는 경우, /workspace/checkpoints/acoustic을 드롭박스로 백업
6. nohup python scripts/train.py --config configs/variance.yaml --exp_name variance & 실행
7.  nohup.out의 업데이트가 없는경우, /workspace/checkpoints/variance를 드롭박스에 백업
8. nohup.out을 드롭박스로 백업.