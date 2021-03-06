# sic assembler
## 1. 전체 구성도 및 설계

<p align="center"><img src="https://user-images.githubusercontent.com/43545606/93041279-1a457980-f687-11ea-8f6c-fd18e468f13a.png" width="600" height="400"></p>

본 어셈블러는 2pass 구조로 pass1, pass2를 거쳐 어셈블리 코드에 대한 목적 코드를 만들어 낸다. Pass1은 어셈블리 코드인 assembly_program.txt를 열어 symbol_table과 intermediate file을 생성한다. 또한 16진수의 형태인 프로그램 길이(programlen), 프로그램 시작 주소를 생성한다. 이렇게 생성한 파일과 변수들을 pass2에 넘겨주어 목적 프로그램(objProgram.txt)와 어셈블리 리스트 파일(listfile.txt)를 최종적으로 생성하게 된다.

## 2. 알고리즘 및 코드 구현

### Pass1
1. 어셈블리 코드를 문서가 끝나기 전까지 한줄씩 읽으면서 레이블 필드를 이름, 연산자, 피연산자로 나누어 저장한다.
2. 읽어온 line의 연산자가 start이면 중간 파일에 주소값 없이 line을 추가한다.
3. 연산자가 end가 아닐 때, comment line이 아니고 symbol이 존재하면 symtab에 symbol과 주소에 대한 데이터를 추가하고 중간 파일에 주소 값과 함께 line을 추가한다.
4. 읽어온 line의 연산자가 optab에 존재하는 경우, locctr에 3만큼을 더해준다.
5. 읽어온 line의 연산자가 어셈블리 지시자인 경우 각 지시자의 종류에 따라 다르게 주소를 지정해줄 수 있도록 구현해주었다.
6. 마지막 줄을 중간 파일에 추가하고 프로그램의 길이를 현재 locctr의 값에서 시작 주소를 뺀 값으로 저장하고 이를 시작 주소와 함께 리턴하여 패스 2로 전달한다.

### Pass2

1. 중간 파일을 문서가 끝나기 전까지 한줄씩 읽으면서 line을 주소, 이름, 연산자, 피연산자로 분리한다.
2. 연산자가 start인 경우, objfile에 헤더 레코드를 작성한다.
3. 연산자가 end인 경우, objfile에 엔드 레코드를 작성한다.
4. line에서 연산자가 그 이외의 것인 경우, optab에 연산자가 존재하고 symtab에 피연산자가 존재하면 “연산자의 기계어”+”피연산자의 주소”가 목적 코드가 된다.
5. opcode stch나 ldch인 경우, 인덱스 주소 지정 방식을 사용하기 때문에 피연산자의 주소의 첫번째 바이트에 8의 값이 더해진 “연산자의 기계어”+”피연산자의 주소”가 목적 코드가 된다.
6. opcode가 rsub인 경우, 목적 코드는 rsub의 기계어에 “0000”을 더한 값이 된다.
7. opcode가 어셈블러 연산자인 경우, 종류에 따라 다르게 목적 코드를 생성할 수 있도록 구현해주었다.
8. (4~7)과정에서 만들어진 목적 코드는 임시 목적 코드 저장소인 temp에 저장되면서 list file에 출력된다.
9. 목적 코드에 텍스트 레코드를 출력할 때는 최대 ie 만큼 출력한 후 이를 넘어가게 되는 경우, 다음 텍스트 레코드에 쓰여야 한다.
