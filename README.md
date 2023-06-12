# 1번 

커맨드 패턴

클라이언트 세션 요청이 왔을때 바로 실행하지 않고 나중에 처리하는 방식
lock을 최소화함,

캡슐화!

언젠가 사용할 Execute 함수를 만들어준다.

```using System;
using System.Collections.Generic;
using System.Text;

namespace Server.Game
{
	public interface IJob
	{
		void Execute();
	}

	public class Job : IJob
	{
		Action _action;

		public Job(Action action)
		{
			_action = action;
		}

		public void Execute()
		{
			_action.Invoke();
		}
	}

	public class Job<T1> : IJob
	{
		Action<T1> _action;
		T1 _t1;

		public Job(Action<T1> action, T1 t1)
		{
			_action = action;
			_t1 = t1;
		}

		public void Execute()
		{
			_action.Invoke(_t1);
		}
	}
```
