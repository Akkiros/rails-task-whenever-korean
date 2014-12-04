# Rails Task Whenever

Rails으로 개발한 어플리케이션에서 주기적으로 실행되는 스크립트 작업을 해야할 일이 있다. 그럴때 Rails task를 사용하면 좋다. whenever은 cron을 쉽게 만들어주는 gem이다.

### Task 생성

```sh
rails g task check_recom_partner run
```

```sh
      create  lib/tasks/check_recom_partner.rake
```

이렇게 task가 하나 생성이 된다. 첫번째 인자는 task의 namespace, 그 이후의 인자들은 task 이름이 된다.

``lib/tasks/check_recom_partner.rake``

```sh
namespace :check_recom_partner do
  desc "TODO"
  task run: environment do
  end
end
```

그럼 우리가 만든 task가(빈껍데기이지만) 잘 돌아가는지 실행시켜보자.

```sh
rake check_recom_partner:run
```

:앞에는 namespace의 이름, 뒤에는 task의 이름을 넣어주면 된다.

run task 안에 아무런 명령어도 없기 때문에 아무일도 발생하지 않는다!

그러면 run task안에 과연 이게 잘 돌아갔는지 확인해보기 위해서 몇가지 코드를 넣어보자.

```sh
namespace :check_recom_partner do
  desc "TODO"
  task run: environment do
    for i in 1..10
      puts i
    end
  end
end
```

1부터 10까지 출력하는 반복문을 만들어보았다. 한번 결과가 잘 출력되는지 테스트해보자

```sh
rake check_recom_partner:run
```

```sh
1
2
3
4
5
6
7
8
9
10
```

우리가 예상한대로 출력이 잘 된다. rails 모델에 대해서도 접근이 가능하다. 

User이라는 모델이 name이라는 속성을 가지고 있다고 가정하면

```sh
namespace :check_recom_partner do
  desc "TODO"
  task run: environment do
    for i in 1..10
      u = User.find_by_id(i)
      u.name if not u.nil?
    end
  end
end
```

1부터 10까지의 숫자를 id로 가지고 있는걸 찾아서 존재하면 그 객체의 이름을 보여주는 아주 간단한 코드다.

이것도 잘 돌아가니 이렇게 task를 활용하면 좋을 것 같다.


### Whenever

Whenever은 cron을 만들기 쉽게 해주는 gem이다. 일정한 문법대로 명령어를 적어주면 그걸 cron에 맞게 변환해준다.

일단 whenever을 시작해보자.

```sh
cd /path/to/your_rails_project
wheneverize .
```

그러면 ``config/schedule.rb``파일이 생성될 것이다.

그러면 우리의 task를 매일 0시 5분에 작동하도록 코드를 작성해보자

``config/schedule.rb``

```sh
every :day, :at => '12:05am' do
  rake "check_recom_partner:run"
end
```

다 작성했으니 cron에 맞게 변환해보자.

```sh
cd /path/to/your_rails_project
whenever
```

rails 프로젝트가 위치한곳에서 whenever 명령어를 입력하면 cron에 맞게 잘 나온다.

```sh
5 0 * * * /bin/bash -l -c 'cd /path/to/project && RAILS_ENV=production bundle exec rake check_recom_partner:run --silent'
```

RAILS_ENV의 경우에는 development 환경이라면 development로 바꿔주면 되고 test환경이면 test로 바꿔주면 된다.

```sh
crontab -e
```
명령어를 입력하여 crontab 설정으로 들어가 저 결과물을 복사/붙여넣기 해주고 저장하면 매일 0시 5분에 자동으로 rails task가 실행 될 것이다.

참고문헌 : https://github.com/javan/whenever 
