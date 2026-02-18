AWS Lambda에 서드파티 라이브러리를 사용하기위해서는 계층을 추가해야한다.

일반적으로는 pip를 이용해서 특정 경로에 라이브러리를 설치 후 이를 lambda를 위한 정해진 경로 구조로 압축을 하면된다.

1. 작업 경로 생성 및 이동
    ```bash
    mkdir my-lambda-layer && cd my-lambda-layer
    mkdir python
    ```
2. --target 옵션을 이용하여 python 폴더 내부에 라이브러리 설치
    ```bash
    pip install <라이브러리이름> -t python/
    ```
3. 용량 최적화를 위한 불필요 캐시 제거
    ```bash
    rm -rf python/*.dist-info python/__pycache__
    ```
4. 압축 파일 생성
    ```bash
    zip -r my-layer.zip python
    ```

위와 같은 과정으로 생성된 zip 파일을 aws lambda의 계층 추가 콘솔에서 업로드하여 계층을 생성한 뒤 해당 계층을 함수에 추가하면된다.

하지만 주의할 점이 있다. 위와 같이 계층을 생성하는 방식은 특정 아키텍처(x86_64, arm64)에 종속되지 않는 순수 python 코드로 이루어진 라이브러리만 가능하다.

c 혹은 rust와 같이 네이티브 기반의 코드가 포함된 라이브러리의 경우 해당 lambda와 동일한 운영체제, 아키텍처를 가지지 않은 라이브러리는 정상 동작하지 않을 확률이 높다.

그래서 해당 라이브러리를 계층으로 생성하여 사용하려면 특별한 방법이 필요하다.

일단 가장 쉬운 방법은 누군가가 미리 빌드한 라이브러리를 사용하는 것이다.

https://github.com/keithrozario/Klayers

또는 pip의 플랫폼 옵션을 통해 manylinux로 빌드된 라이브러리를 설치하는 것이다.

https://repost.aws/ko/knowledge-center/lambda-python-package-compatible