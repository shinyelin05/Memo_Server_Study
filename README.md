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

---

# 2번

lock을 없애면 오류가 나는데, 다 없애준다.

Push를 해주면 Job에서 알아서 푸쉬를 해준다.

	public void Push(Action action) { Push(new Job(action)); }
		public void Push<T1>(Action<T1> action, T1 t1) { Push(new Job<T1>(action, t1)); }
		public void Push<T1, T2>(Action<T1, T2> action, T1 t1, T2 t2) { Push(new Job<T1, T2>(action, t1, t2)); }
		public void Push<T1, T2, T3>(Action<T1, T2, T3> action, T1 t1, T2 t2, T3 t3) { Push(new Job<T1, T2, T3>(action, t1, t2, t3)); }

		public void Push(IJob job)
		{
			lock (_lock)
			{
				_jobQueue.Enqueue(job);
			}
		}
		
		계속해서 에러가 나는 부분들을 해결하면서 해야한다. 

커맨트 패턴으로 일단 해두면서 진행하면 된다.

무한 루프는 아니다!

EnterGame은 인자를 this로 등등.. 바꿔준다.

---

# 3번

클라이언트를 만들고 JobTimer처럼 시스템적인 게임이 필요하다.

어떻게 하면 효율적으로 돌아가야할까? 생각하기

![image](https://github.com/shinyelin05/Memo_Server_Study/assets/77713669/dcc5d45b-bb5d-49b9-9e65-0cbc1a8e72b9)

Update도 주기 별로 어떻게 해야할지 생각해야한다

메인 쓰레드를 고려해봐야한다.

그리고 Push와 Flush하는 것을 나눈다.

---

# 4번

간단하게 테스트를 해보고 버그를 고치는 작업으로 진행 할 것이다.

널 체크를 해서 몬스터가 죽을 때의 버그를 고친다. 

 	 else if (type == GameObjectType.Monster)
			{
				Monster monster = gameObject as Monster;
				_monsters.Add(gameObject.Id, monster);
				monster.Room = this;

				Map.ApplyMove(monster, new Vector2Int(monster.CellPos.x, monster.CellPos.y));
			}
			else if (type == GameObjectType.Projectile)
			{
				Projectile projectile = gameObject as Projectile;
				_projectiles.Add(gameObject.Id, projectile);
				projectile.Room = this;
			}
			
			// 타인한테 정보 전송
			{
				S_Spawn spawnPacket = new S_Spawn();
				spawnPacket.Objects.Add(gameObject.Info);
				foreach (Player p in _players.Values)
				{
					if (p.Id != gameObject.Id)
						p.Session.Send(spawnPacket);
				}
			}
