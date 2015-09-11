title: CE3备忘录3--暗杀（Stealthkill）bug的简单处理
date: 2014-09-15 05:53:58
tags: [Code,Game]
categories: CE3
---
关于备忘录：
这里主要是自己记录一些CE3(Cryengine3)的常用方法，包括代码编写以及API功能，如果有什么问题感谢指出，这是我的邮箱albstein2@gmail.com

---

在使用CE3时遇到点问题，当从背后靠近敌人时，我们使用近战攻击时敌人会直接死亡，但是我们在之后却无法移动，

断点调试后，发现在这种情况下实际是触发了引擎中自带的暗杀系统，而在这一部分中我们并没有提供相应的暗杀动作在其中，这就会触发这个bug（当然具体的问题我还没有定位到）

下面为有同样问题的开发者们提供一个可行的解决办法，就是直接把调用暗杀的部分去掉（既然没有动画，那我们就不用它了）

下面是我通过调试找到的代码段：
```c
bool CPlayerInput::OnActionSpecial(EntityId entityId, const ActionId& actionId, int activationMode, float value)
{
	if (CallTopCancelHandler())
	{
		return false;
	}
 
	const SInteractionInfo& interactionInfo = m_pPlayer->GetCurrentInteractionInfo();

	if (activationMode == eAAM_OnPress)
	{
		if (interactionInfo.interactionType == eInteraction_Stealthkill)
		{
			m_pPlayer->AttemptStealthKill(interactionInfo.interactiveEntityId);//这里调用的暗杀
		}
		else if (interactionInfo.interactionType == eInteraction_LargeObject)
		{
			if(!m_pPlayer->GetLargeObjectInteraction().IsBusy())
			{
				m_pPlayer->EnterLargeObjectInteraction(interactionInfo.interactiveEntityId); //always use power kick in MP
			}

		}
	}
 
	return false;//如方法名，这里触发的是特殊动作,前面两段代码都没有执行的话，就会进入正常的近战流程
}
```

代码可以通过搜索的方法简单的定位到，之后我们只需将调用暗杀的哪行注释掉就行了

```C
bool CPlayerInput::OnActionSpecial(EntityId entityId, const ActionId& actionId, int activationMode, float value)
{
	if (CallTopCancelHandler())
	{
		return false;
	}
 
	const SInteractionInfo& interactionInfo = m_pPlayer->GetCurrentInteractionInfo();

	if (activationMode == eAAM_OnPress)
	{
		if (interactionInfo.interactionType == eInteraction_Stealthkill)
		{
			//我们把它注释掉
			//m_pPlayer->AttemptStealthKill(interactionInfo.interactiveEntityId);
		}
		else if (interactionInfo.interactionType == eInteraction_LargeObject)
		{
			if(!m_pPlayer->GetLargeObjectInteraction().IsBusy())
			{
				m_pPlayer->EnterLargeObjectInteraction(interactionInfo.interactiveEntityId); //always use power kick in MP
			}

		}
	}

	return false;
}
```

好的，这样当我们从敌人背后悄悄靠近的时候就可以用近战愉快的揍他了
