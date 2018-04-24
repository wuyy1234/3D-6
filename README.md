# 3D-6
## 改进飞碟（Hit UFO）游戏
### 游戏内容要求：
* 按 adapter模式 设计图修改飞碟游戏  
* 使它同时支持物理运动与运动学（变换）运动  

#### 界面(通过按钮切换两种模式): 
 <img src="http://imglf5.nosdn.127.net/img/Z281REhERnhNZlhqRGlHd3BXK2RxQjVmVmVmVHZMVnQyVGZXQkpKZU14TUpMV0MwMWpwRlp3PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0"  />  
  <img src="http://imglf6.nosdn.127.net/img/Z281REhERnhNZlhqRGlHd3BXK2RxRGhhcmVmWHltdHRTcURRMGlyMUw2bkRxZkh0a3V4ZUVRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0"  />
    <img src="http://imglf5.nosdn.127.net/img/Z281REhERnhNZlhqRGlHd3BXK2RxTnNHT3BIWWxsWDA2WmJBQVBCc3RDQWVUbkdZVG1KdVB3PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0"  />

   <img src="http://imglf5.nosdn.127.net/img/Z281REhERnhNZlhqRGlHd3BXK2RxSDY2WThtMG5uSWw2eTlUK2pRRE81Z250STVqdnQ1QjRRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0"  />
<img src="http://imglf5.nosdn.127.net/img/Z281REhERnhNZlhqRGlHd3BXK2RxSVRoUThmVG40Yzk1VFRxRGcxaXFPemdtYk45dW0zc2l3PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0"  />  

#### 原架构：  
 <img src="http://imglf6.nosdn.127.net/img/Z281REhERnhNZlhqRGlHd3BXK2RxRE5Qc1lhREpCbEkrcENTZkh3aHo1dlVmaXdEODUrM2J3PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0"  />
 <img src="http://imglf5.nosdn.127.net/img/Z281REhERnhNZlhqRGlHd3BXK2RxT1BUOHdKSVFNOGtLL0dLTiszYkI4Zkg1a0Y5Y3lBWEZRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0"  />  
 
#### 修改后的架构： 
  <img src="http://imglf5.nosdn.127.net/img/Z281REhERnhNZlhqRGlHd3BXK2RxS3AzN0hMSU8rQnk1VTg3YlRWdFZrcHFnU2tVRmZ0OGNBPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0"  />  
  <img src="http://imglf5.nosdn.127.net/img/Z281REhERnhNZlhqRGlHd3BXK2RxSkdVNTEvQWtsWmx1cWpjbDd5YW8yR1RKaDBmbXMrUWtnPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0"  />  
  <img src="http://imglf6.nosdn.127.net/img/Z281REhERnhNZlhqRGlHd3BXK2RxRC90YzgrbXlhTUw1SHhIZFd6QnVIK2Y3NkhVVERVNFJ3PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0"  />

 
#### 主要的改进方案：采用Adapter模式  ，将动作ThrowUFO变成两种实现方式  
物理变化：  
```  
public void throwUFO (List<GameObject> usingUFO){
		_usingUFO=usingUFO;
		Scene.getScene().reset(Controller.getController().round);
		for(int i=0;i<_usingUFO.Count;i++){//向图中扔飞碟,设置方向，大小，颜色等属性
			usingUFO [i].transform.position = throwPosition;
			Rigidbody rigibody;
			rigibody = usingUFO[i].GetComponent<Rigidbody>();
			rigibody.WakeUp();
			rigibody.useGravity = true;
			rigibody.AddForce(throwDirection, ForceMode.Impulse);
		}
	}
``` 

另一种transform变化：  
``` 
public class CCThrowUFO : SSAction{
	private Vector3 throwPosition;
	private Vector3 throwDirection;
	private float gravity=9.8f;

	public static CCThrowUFO GetSSAction()  
	{  
		CCThrowUFO action = ScriptableObject.CreateInstance<CCThrowUFO>();  
		return action;  
	}  
	void Start () {

		throwDirection = new Vector3(3f, 10f, 15f);
		throwPosition = new Vector3 (-2f, 1f, 0);
		this.gameobject.transform.position = throwPosition;
	}
	void Update () {
		Debug.Log ("CCActionManager  throw UFO  destroy: "+this.destroy);
		Vector3 temp=new Vector3(Time.deltaTime * throwDirection.x , 
			Time.deltaTime*(throwDirection.y-gravity*Time.deltaTime) ,Time.deltaTime*throwDirection.z);
		this.gameobject.transform.position += temp;

		if(this.gameobject.transform.position.y < -30) {
			this.destroy = true;  
			this.callback.SSActionEvent(this);  
		}
	
	}
}
```

#### 源码：  

ControllerMono.cs
``` 
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using com.UFO;

namespace com.UFO{
	public class Controller  {
		public int round{ get; set;}//从1开始
		public int choosePhyActOrCCAct{get;set;}//0为phy，1为CC

		public static Controller _Controller;
		private List<GameObject> usingUFOs;
		private List<GameObject> uselessUFOs;


		public List<GameObject> getUsingUFOs(){
			return usingUFOs;
		}
		public List<GameObject> getUselessUFOs(){
			return uselessUFOs;
		}

		public static Controller getController(){
			if (_Controller == null){
				_Controller = new Controller();
				_Controller.usingUFOs=new List<GameObject>();
				_Controller.uselessUFOs = new List<GameObject> ();
			}
			return _Controller;
		}

		public void addScore(int addNum){
			model.getModel ().addScore (addNum);
		}
		public void subScore(int subNum){
			model.getModel ().subScore (subNum);
		}
		public void throwUFO(int num){
			//先从工厂里面拿飞盘
			usingUFOs=UFOFactory.getFactory().creatUFOs(num);
			Scene.getScene ().throwUFO (usingUFOs);

		}
		public void deleteUFO(GameObject UFO){
			//Debug.Log ("controller delete");
			UFOFactory.getFactory().deleteUFO(UFO);
			Scene.getScene().deleteUFO(UFO);
		}
	}

	public class ControllerMono:MonoBehaviour{
		public GameObject UFOPrefab;
		public UFOFactory _UFOFactory;
		public static CCActionManager _CCActionManager;

		void Start () {
			_UFOFactory = UFOFactory.getFactory();
			_UFOFactory.UFOprefab = this.UFOPrefab;
			_CCActionManager = gameObject.AddComponent<CCActionManager> ();

		}
		void Update () {
				for (int i = 0; i < _UFOFactory.getUsingUFOs().Count; i++) {
				if (_UFOFactory.getUsingUFOs()[i].transform.position.y < -30) {//判断出界,回收
					//Debug.Log("")
					model.getModel().subScore(10);
					Controller.getController().deleteUFO(_UFOFactory.getUsingUFOs()[i]);

				}
			}
		}
		public static CCActionManager getCCActionManager(){
			return _CCActionManager;
		}
	}

} 

```

model.cs  

``` 
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using com.UFO;

public class model{//进行成绩等游戏逻辑处理

	public int Score{ get;set;}
	public int Status{ get;set;}//-1为输，1为赢，0表示继续


	public static model _model;
	public static model getModel(){
		if (_model == null){
			_model = new model();
		}
		return _model;
	}
	public void resetModel(){
		Score = 0;
		Status = 0;
	}

	public void addScore(int addNum){
		//model.getModel().Score = addNum+model.getModel().Score;
		this.Score+=addNum;
		//Debug.Log ("score: " + this.Score);


		if (model.getModel ().Score >= 100) {//胜利
			Status = 1;
			Scene.getScene ().showChangeStatus ();
		} else if (model.getModel ().Score >= 0) {
			Status = 0;
			Scene.getScene ().showChangeStatus ();
		}
	}
	public void subScore(int subNum){
		Score -= subNum;

		if (Score < 0) {//失败
			Status = -1;
			Scene.getScene ().showChangeStatus ();
		}

	}


}

```

Interact.cs  
``` 
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using com.UFO;

public class Interact : MonoBehaviour {


	private GUIStyle fontStyle;
	private float time;
	model _model;
	void Start () {
		
		fontStyle=new GUIStyle();
		fontStyle.fontSize = 40;
		//_scene = Scene.getScene ();
		_model = model.getModel ();
		_model.resetModel ();
	}
	void OnGUI(){
		if (GUI.Button (new Rect (Screen.width / 2 - 350, Screen.height / 2 - 275, 200, 50), "choose physical action")) {
			Controller.getController ().choosePhyActOrCCAct = 0;
		} else if (GUI.Button (new Rect (Screen.width / 2 +150, Screen.height / 2 - 275, 200, 50), "choose CC action")) {
			Controller.getController ().choosePhyActOrCCAct = 1;
		}

		if (Controller.getController ().choosePhyActOrCCAct == 0) {
			GUI.Label (new Rect (Screen.width / 2 - 150, Screen.height / 2 + 150, 100, 50), "physical model",fontStyle);
		} else {
			GUI.Label (new Rect (Screen.width / 2 - 150, Screen.height / 2 + 150, 100, 50), "CCAction model",fontStyle);
		}

		GUI.Label (new Rect (Screen.width / 2-100, Screen.height / 2-300, 100, 50),"score: "+ _model.Score.ToString(),fontStyle);
		if ( Scene.getScene().getshowChangeStatusEnable()) {
			if (model.getModel().Status == -1) {
				GUI.Label (new Rect (Screen.width / 2, Screen.height / 2, 100, 50), "fail", fontStyle);
			} else if (model.getModel().Status == 1) {
				GUI.Label (new Rect (Screen.width / 2, Screen.height / 2, 100, 50), "win", fontStyle);
			}
		}

	}

	void Update () {
		time += Time.deltaTime;
		if (time >2.5&&UFOFactory.getFactory().getUsingUFOs().Count==0) {//在全部UFO都被回收之后才发射  最快每2.5秒发射一个
			time=0;
			Controller.getController ().throwUFO (1);
		}
		if (Input.GetMouseButtonDown (0)) {
			Ray ray = Camera.main.ScreenPointToRay (Input.mousePosition);
			RaycastHit hit;
			if (Physics.Raycast (ray, out hit) && hit.collider.gameObject.tag == "UFO") {
				//Debug.Log ("click UFO");
				Controller.getController ().deleteUFO (hit.collider.gameObject);

				Controller.getController().addScore(10);
			}
		
		}

	}
}

```


Scene.cs  
``` 
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using com.UFO;


namespace com.UFO{
	public class Scene{
		public static Scene _Scene;
		public List<GameObject> _usingUFO;


		private bool showChangeStatusEnable;
		private int _score;
		//private int _status;
//		private Vector3 throwPosition;
		//private double throwSpeed;
	//	private Vector3 throwDirection;
		private int UFOspeed;

		public bool getshowChangeStatusEnable(){
			return showChangeStatusEnable;
		}

		public static Scene getScene(){
			if (_Scene == null){
				_Scene = new Scene();
				_Scene._usingUFO= new List<GameObject>();
			}
			return _Scene;
		}
		public void reset(int round){//根据回合数重置飞边扔出的各种参数
			//throwSpeed=1;
		//	throwDirection = new Vector3(3f, 10f, 15f);
		//	throwPosition = new Vector3 (-2f, 1f, 0);
		}

		public void throwUFO(List<GameObject> usingUFO){//扔飞碟
			if (Controller.getController ().choosePhyActOrCCAct == 0) {//0为phy，1为CC
				PhysicsActionManager.getPhysicsActionManager ().throwUFO (usingUFO);
			} else {
			//调用CCActionManager

				ControllerMono.getCCActionManager().throwUFO(usingUFO);
			}


			/*_usingUFO=usingUFO;
			getScene().reset(Controller.getController().round);
			for(int i=0;i<_usingUFO.Count;i++){//向图中扔飞碟,设置方向，大小，颜色等属性

				//usingUFO [i].transform.position =new Vector3(throwPosition.x,throwPosition.y,throwPosition.z);
				usingUFO [i].transform.position = throwPosition;
				Rigidbody rigibody;
				rigibody = usingUFO[i].GetComponent<Rigidbody>();
				rigibody.WakeUp();
				rigibody.useGravity = true;
				//rigibody.AddForce(throwDirection* Random.Range((float)throwSpeed * 5, (float)throwSpeed* 8) / 5, ForceMode.Impulse);
				rigibody.AddForce(throwDirection, ForceMode.Impulse);
				//rigibody.velocity = throwDirection;
				//usingUFO[i].SetActive(true);
				//Debug.Log ("throwUFO");

			}*/
		}
		public void  deleteUFO(GameObject UFO){
			UFO.GetComponent<Rigidbody> ().Sleep ();
			UFO.GetComponent<Rigidbody> ().useGravity = false;
			UFO.transform.position = new Vector3 (0f, 20f, 0f);

		}


		public void showChangeStatus(){
			
			showChangeStatusEnable = true;
		}
	}
}
```
CCThrowUFO.cs  
``` 
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using com.UFO;

public class CCThrowUFO : SSAction{
	private Vector3 throwPosition;
	private Vector3 throwDirection;
	private float gravity=9.8f;

	public static CCThrowUFO GetSSAction()  
	{  
		//Debug.Log ("CreateInstance<CCThrowUFO>");
		CCThrowUFO action = ScriptableObject.CreateInstance<CCThrowUFO>();  
		return action;  
	}  

	// Use this for initialization
	void Start () {

		throwDirection = new Vector3(3f, 10f, 15f);
		throwPosition = new Vector3 (-2f, 1f, 0);
		this.gameobject.transform.position = throwPosition;
	}
	
	// Update is called once per frame
	void Update () {
		Debug.Log ("CCActionManager  throw UFO  destroy: "+this.destroy);
		Vector3 temp=new Vector3(Time.deltaTime * throwDirection.x , 
			Time.deltaTime*(throwDirection.y-gravity*Time.deltaTime) ,Time.deltaTime*throwDirection.z);
		this.gameobject.transform.position += temp;

		if(this.gameobject.transform.position.y < -30) {
			this.destroy = true;  
			this.callback.SSActionEvent(this);  
		}
	
	}
}

```

SSAction.cs

``` 
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using com.UFO;

public enum SSActionEventType : int { Started, Competeted }  

public interface ISSActionCallback  
{  
	void SSActionEvent(SSAction source, SSActionEventType events = SSActionEventType.Competeted,  
		int intParam = 0, string strParam = null, Object objectParam = null);  
}  

public class SSAction : ScriptableObject  
{  
	public bool enable = true;  
	public bool destroy = false;  

	public GameObject gameobject { get; set; }  
	public Transform transform { get; set; }  
	public ISSActionCallback callback { get; set; }  

	protected SSAction() { }  

	public virtual void Start()  
	{  
		//throw new System.NotImplementedException();  
	}  

	public virtual void Update()  
	{  
		//throw new System.NotImplementedException();  
	} 
	public void reset(){
		enable = false;
		destroy = true;
		gameobject = null;
		transform = null;
		callback = null;
	}
}  

```

SSActionManager.cs

``` 
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using com.UFO;

public class SSActionManager : MonoBehaviour {
	private Dictionary<int, SSAction> actions = new Dictionary<int, SSAction>();  
	private List<SSAction> waitingAdd = new List<SSAction>();  
	private List<int> waitingDelete = new List<int>();  

	// Use this for initialization  
	void Start()  {  
	}  

	// Update is called once per frame  
	protected void Update()  
	{  
		//Debug.Log ("waitingAdd: " + waitingAdd.Count);
		foreach (SSAction ac in waitingAdd) {
			actions[ac.GetInstanceID()] = ac;  

		} 
		waitingAdd.Clear();  

		foreach (KeyValuePair<int, SSAction> kv in actions)  
		{  
			SSAction ac = kv.Value;  
			if (ac.destroy)  
			{  
				waitingDelete.Add(ac.GetInstanceID());  
			}  
			else if (ac.enable)  
			{  
				//Debug.Log("")
				ac.Update();  
				//ac.FixedUpdate();  
			}  
		}  

		foreach (int key in waitingDelete)  
		{  
			SSAction ac = actions[key]; 
			actions.Remove(key); 
			DestroyObject(ac);  
		}  
		waitingDelete.Clear();  
	}  

	public void RunAction(GameObject gameobject, SSAction action, ISSActionCallback manager)  
	{  
		
		action.gameobject = gameobject;  
		action.transform = gameobject.transform;  
		action.callback = manager;  
		action.enable = true;
		waitingAdd.Add(action); 
		Debug.Log ("action: " + action);
		action.Start();  
	}  
}  
```

CCActionManager.cs  
``` 
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using com.UFO;

public class CCActionManager :  SSActionManager,ISSActionCallback{
	public  CCThrowUFO _CCThrowUFO;
	//private  CCActionManager _CCActionManager;



	public void throwUFO (List<GameObject> usingUFO){
		_CCThrowUFO = CCThrowUFO.GetSSAction ();
		foreach (GameObject UFO in usingUFO) {
			
			this.RunAction (UFO, _CCThrowUFO, this);
		}

	}


	public void SSActionEvent(SSAction source, SSActionEventType events = SSActionEventType.Competeted,  
		int intParam = 0, string strParam = null, Object objectParam = null)  {
	}
}


```

IActionManager.cs
``` 
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using com.UFO;

public interface IActionManager {
	void throwUFO (List<GameObject> usingUFO);
}

```





