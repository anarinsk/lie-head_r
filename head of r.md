
# R 사용자를 위한 효과적인 패키지 및 폴더 관리 

* 공동 게시: https://danbi-ncsoft.github.io/etc/2018/08/14/head-of-r.html 

## 들어가며 

다음 질문에 답해보자.

* 원시 데이터(raw data)를 저장할 폴더를 효과적으로 두는 방법은? 
* 코드 내에서 흩어진 다른 코드나 데이터를 읽어와야 할 때에는 어떻게 읽어오면 좋을까? 

혼자 작업한다면, 이런 것들이 크게 문제가 되지 않을 수도 있다. 어차피 내가 하는 것이니까. 그런데 사실 혼자 할 때도 문제가 될 수 있다. 한 달 전의 나와 지금의 내가 같다고 할 수 있는가? 나는 보통 한 달 전에 했던 일을 잘 기억하지 못한다. 

이럴 때를 대비해서 보통 주석 등을 통해 문서화를 한다. 그런데 때로는 문서를 보고도 잘 파악이 안되는 경우도  있다. 이럴 때는 문서를 이해하기 위한 '메타' 문서를 만들어 두어야 할까? 이런 식으로 메타를 무한 반복할 수는 없는 노릇이다. 

누가 코드 혹은 프로젝트를 읽더라도 동일한 환경에서 그대로 실행할 수 있는 것이 이상적이다. 가장 좋은 방법은 컴퓨터 하드디스크의 이미지 떠서 복제하듯이 잘 돌아간 환경을 그대로 떠서 이 '냉동 제품'을 그대로 쓰게 하는 것이다. 요즘은 docker를 이용하면 이러한 방식으로 내 환경을 복제하고 공유하는 것이 가능하다. 하지만 아직은 살짝 높아 보이는 진입 장벽이 있다. 게다가   docker를 동원할 정도로 대단한 작업이 아니라면 뭔가 배보다 배꼽이 더 큰 느낌이다.  

보통 내 코드를 다른 사람과 공유할 때 문제가 되는 것은 무엇일까? 

* 내 코드가 인용하는 폴더(디렉토리)들이 다른 사람의 구현 환경에서도 제대로 인용될까? 
* 내 코드가 활용한 패키지들이 다른 구현 환경에서도 제대로 설치되어 있을까? 

이 두 가지 문제를 비교적 손쉽게 해결할 수 있는 방법을 알아보자. 

## 패키지 설치 및 유지 보수 

패키지 설치 문제 때문에 나도 보통 코드 앞에 아래와 같은 메타 코드를 두었다. 

```r
install_load_package <- function(packages_v){
  #
  new.packages <- packages_v[!(packages_v %in% installed.packages()[,"Package"])]
  if(length(new.packages) > 0) { install.packages(new.packages) } 
  lapply(packages_v, require, character.only = TRUE)
  invisible(capture.output())
  cat("--------------- \n")
  lapply(packages_v, function(x) {cat(x, "are loaded! \n")})
  cat("--------------- \n")
  #
}
#
```

이 함수는 이렇게 작동한다. 입력으로 받은 패키지 스트링에 대해서 해당 패키지가 컴퓨터에 깔려 있으면 설치하지 않고 로드만 한다. 만일 깔려 있지 않으면 패키지들을 설치한 후 로드한다. 그런데, 이 패키지의 가장 큰 문제는 오직 CRAN(Comrehensive R Archive Network)에 있는 패키지만 가져올 수 있다는 것이다. 아시다시피 요즘 최신 패키지가 가장 먼저 배포되는 곳 그리고 기존 패키지의 최신 버전이 가장 먼저 등장하는 곳은 github이 아닌가? 아마도 비슷한 문제를 겪은 사람들이 많았는지 최근에 좋은 패키지가 나왔다. 

[https://github.com/DesiQuintans/librarian](https://github.com/DesiQuintans/librarian)

```{r}
librarian::shelf(dplyr, DesiQuintans/desiderata, purrr)
                   ↑         ↑                    ↑
                  CRAN     GitHub                CRAN

# All downloaded, installed, and attached.
```

librarian 패키지의 장점은 다음과 같다. 

* CRAN과 github을 모두 포괄한다. 
* 패키지 업데이트가 있을 경우도 새로 설치해준다. 
* 그리고 quote 처리를 할 필요도 없다. 

## 일관되게 폴더 인용하기 

폴더의 기준점을 잡기 위해 당신은 어떤 방법을 쓰는가? 가장 흔하게는 `setwd()` 명령어로 디폴트 폴더를 직접 지정하는 것이다. 그런데 이렇게 지정을 하고 나면  R 세션 전체가 영향을 받는다. 왠지 찝찝하지 않은가? 게다가 OS를 옮겨다니며 작업한다면, OS 차이에 따른 미묘한 이슈가 발생할 수도 있다.  

### proj in RStudio 
RStudio를 쓴다면 이럴 때 project 기능을 활용하면 좋다. 작업 중인 폴더의 이름이 `YOUR_PROJECT_NAME`이었다고 하자. 이 폴더 내에 프로젝트를 생성하면 해당 폴더에 `YOUR_PROJECT_NAME.Rproj`라는 파일이 생긴다. 이 파일을 실행하면 RStudio에서 프로젝트가 열린다. 그리고 `getwd()`를 실행해보자. 프로젝트 생성은 

RStudio &rarr; File &rarr; New Project 를 실행하면 아래와 같은 화면이 뜬다. 

![](https://github.com/anarinsk/lie-head_r/blob/master/assets/etc/head-of-r/preamble_1.png?raw=true)

새 폴더에 프로젝트를 생성할 경우에는 "New Directory"를 기존 폴더를 프로젝트로 잡으려면 "Existing Directory"를 선택하면 된다. "Version Control"은 당분간 무시하도록 하자. 

해당 프로젝트가 위치한 최상위 폴더가 기본 폴더로 잡힌다. 이제 코드 안에서 걱정 없이 하위 폴더를 인용할 수 있다. 

한번 로딩했던 프로젝트는 그림에서 보듯이 RStudio에서 빠르고 쉽게 이동할 수 있다. 

![](https://github.com/anarinsk/lie-head_r/blob/master/assets/etc/head-of-r/preamble_2.png?raw=true)

### here 패키지

RStudio의 프로젝트와 짝꿍으로 쓰기 좋은 패키지가 [here](https://github.com/r-lib/here) 패키지다.  패키지 페이지에서 보듯이 기본 폴더를 기준으로 하위 폴더 인용을 쉽게 도와준다. 

```r
library(here)
#> here() starts at /home/travis/build/r-lib/here
here::here()
#> [1] "/home/travis/build/r-lib/here"
here::here("DESCRIPTION")
#> [1] "/home/travis/build/r-lib/here/DESCRIPTION"
here::here("R", "here.R")
#> [1] "/home/travis/build/r-lib/here/R/here.R"
```

## 조금 더 심각한 R의 프로젝트 관리 도구들 

위 세 가지 조치(librarian 패키지, RStudio의 프로젝트 기능, here 패키지) 정도면 개인 차원의 프로젝트는 감당이 될 것이라고 본다. 하지만 더욱 복잡하고 골치 아픈 프로젝트를 처리해야 한다면, 보다 전문적인 툴도 준비가 되어 있다. 

* https://github.com/ropensci/drake
* https://github.com/ropensci/rrrpkg

## 마치며 

사실 여기에 git, github 이야기는 쓰지 않았다. 이것은 별도의 주제로 다루어야 하는 내용이기도 하고 버전 관리 이야기까지 섞으면 초점이 흐려질 수 있기 때문이다. 여기에 git까지 활용할 수 있다면 금상첨화일텐데, 이는 기회가 되면 다음에 다뤄보도록 하자. 

## 참고 자료 

* https://www.tidyverse.org/articles/2017/12/workflow-vs-script/
* https://swcarpentry.github.io/r-novice-gapminder/02-project-intro/
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjE0Njg5MTE3NSwxNDAyMzA4NTQ1XX0=
-->