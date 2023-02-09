<script>
	import Header from "./Header.svelte";
	import axios from "axios";
	import {ApiHost} from "./Config.svelte"

	let m_CreateTradeIng = false;
	let m_jumpToPayPage = false;
	let m_CurrencyList = [];
	let m_AgreementList=[];
	let m_CurrencyTypeImage = false;

	// ----- Put
	let m_TiXianCount = 0;
	let m_MoneryAddr = "";

	let m_HaveCostInfo = false;
	let m_Cost_High = 2;
	let m_Cost_Low = 1.3;
	let m_Cost_Middle = 1.36;
	let m_Select_Cost_Index = 1;
	let m_CurrencyRate = false;
	let m_CurrencyAlias = "";
	let m_ErrorText = false;

	let g_currencyId = false;
	let g_argreementId = false;
	let g_isSubmit = false;
	// -----

	function CreateTrade() {
		if(m_CreateTradeIng | m_jumpToPayPage) return;

		m_CreateTradeIng = true;

		axios.get(`${ApiHost}/api/UserPutUSDT?code=Dafk8VFnHuPD5OWp-bXNOZb7oElZ8khhOqK5GP7UjEFdAzFu7yD0lQ==`)
			.then(response => {
				var data = response.data;
				if(data.code == 0)
				{
					var payUrl = data.data.Url;
					if(payUrl == undefined)
						throw new error("UrlError.");
					else
					{
						m_jumpToPayPage = true;
						window.location.href = data.data.Url;
					}
				}
				else
				{
					throw new error("创建订单失败，请稍后再试.");
				}
			})
			.catch(_ => {
				alert("创建订单失败，请稍后再试.");
			})
			.finally( () => {
				m_CreateTradeIng = false;
			})
	}

	function GetCurrencyList()
	{
		axios.get(`${ApiHost}/api/GetCurrency?code=ATgK4cHiiCrlL2u8SJ1_lSbUEz8SiEUqBJc2nmpPB_dQAzFuYC_8Ag==`)
		.then( (response) => {
			if(response.data.code == 0)
			{
				var clist = [];
				var alist = [];

				response.data.data.currency_list.forEach( p => {
					if(response.data.data.agreement_list[p.id].length > 0){
						clist.push(p);
					}
				});
				m_CurrencyList = clist;
				m_AgreementList = response.data.data.agreement_list;


				CurrencyChanged(m_CurrencyList[0].id);
			}
			else 
				throw new error("GetCurrencyError");
		}).catch(_=>{

		}).finally(()=>{

		});
	}

	function CurrencyChanged(currencyId)
	{
		var currencyId = parseInt(currencyId);
		var argreementId = m_AgreementList[currencyId][0].id;

		m_CurrencyList.forEach(a => {
			if(a.id == currencyId){
			 	m_CurrencyTypeImage = a.thumb;
				m_CurrencyAlias = a.alias;
			}
		});

		// GetCost
		GetCost(argreementId, currencyId);
	}

	function GetCost(argid, cryId){
		var agreement_id=argid;
		var currency_id=cryId;

		m_HaveCostInfo = false;

		axios.get(`${ApiHost}/api/GetCost?code=g0QWnfVGC0wMZBS7lOklB5sQLdh3oriuWaltLBDuISDnAzFuf4VTIA==&agreement_id=${argid}&currency_id=${cryId}`)
		.then( (response) => {
			if(response.data.code != 0) 
				throw new Error("GetCostError");
			
			m_Cost_High = response.data.data.high;
			m_Cost_Middle = response.data.data.middle;
			m_Cost_Low = response.data.data.low;

			g_argreementId = argid;
			g_currencyId = cryId;

			m_HaveCostInfo = true;
		}).catch(err => {
			alert("CostError");
			console.error(e);
			m_HaveCostInfo = false;
		});

		// Get Rate
		axios.get(`${ApiHost}/api/GetRate?code=dpL8RfKX2i0Y1xzMgDIJot6YG-Ex4JgfsMgM_NNjg0joAzFuinSOOg==&currency_id=${cryId}`).then(res=>{
			var rate = res.data.data.rate;
			m_CurrencyRate = rate;
		}).catch(err => {
			console.log(err);
		});
	}

	function OnSubmit() {
		if(g_isSubmit) return;

		if(m_MoneryAddr == "") {
			m_ErrorText ="钱包地址不能为空";
			return;
		}
		else if(m_Select_Cost_Index == 0) {
			m_ErrorText = "旷工费率不能为空";
			return;
		}
		else if(isNaN(m_TiXianCount)) {
			m_ErrorText = "提币数量必须是数值";
			return;
		}
		else if(m_TiXianCount<=0){
			m_ErrorText = "提币数量必须大于0";
			return;
		}
		else if(!g_argreementId) {
			m_ErrorText = "币种协意错误";
			return;
		}else if(!g_currencyId) {
			m_ErrorText = "币种错误";
			return;
		}

		//public class LanchUserData {
        //    public int agreement_id {get;set;}
        //    public int currency_id {get;set;}
        //    public string address {get;set;}
        //    public int cost_type {get;set;}
        //    public int amount {get;set;}
        //}

		var postData = {
			agreement_id : g_argreementId,
			currency_id: g_currencyId,
			address : `"${m_MoneryAddr}"`,
			cost_type: m_Select_Cost_Index,
			amount: `"${m_TiXianCount}"`
		}

		g_isSubmit = true;

		axios.post(`${ApiHost}/api/UserLaunch?code=quBuwLVPyoYYVyryAkbzg3y75HPPuC9gCvHpqyzOfNAKAzFu0ymGUg==`, postData)
		.then(res => {
			m_ErrorText = JSON.stringify(res.data);
		})
		.catch(err => {
			m_ErrorText = err.message;
			console.error(err);
		})
		.finally(() => {
			g_isSubmit = false;
		});
	}

	GetCurrencyList();
</script>

<main>
	<Header></Header>

	<div class="card" style="margin: 20px;">
		<div class="card-body d-grid">
			<button class="btn btn-primary btn-block" on:click={CreateTrade}>
				{#if m_CreateTradeIng}
				<span class="spinner-border spinner-border-sm"></span>
				正在创建订单..
				{:else}
				测试充值
				{/if}
			</button>
		</div>
	</div>
	
	<div class="card" style="margin: 20px;">
		<div class="card-body d-grid">
 
			<form >
				<label for="sel1" class="form-label">提款币种</label>
				{#if m_CurrencyTypeImage != false}
				<img src="https://runzhibao.cc/{m_CurrencyTypeImage}" alt="logo" width="42" height="42" class="img-thumbnail">
				<br/>
				<br/>
				{/if}
				<select on:change={(e) => CurrencyChanged(e.target.value)} title="Select" class="form-select selectpicker">
					{#each m_CurrencyList as {alias,thumb,id}}
					<option value={id} data-thumbnail="https://runzhibao.cc/{thumb}">{alias}</option>
					{/each}
				</select>
			</form>

			{#if m_TiXianCount == 0}
			<input type="number"  bind:value={m_TiXianCount} min="0" max="999" maxlength="6" class="form-control" id="useraddr" placeholder="提币数量" name="useraddr"/>
			{:else}
			<input type="number" bind:value={m_TiXianCount} min="0" max="999" maxlength="6" class="form-control" id="useraddr" placeholder="提币数量" name="useraddr"/>
			{/if}

			<input type="text" bind:value={m_MoneryAddr} class="form-control" id="useraddr" placeholder="钱包地址" name="useraddr"/>
			
			{#if m_CurrencyRate != false}
			
			<strong style="text-align: left;">汇率: 1 {m_CurrencyAlias} = {m_CurrencyRate} USDT-TRC20</strong>
			{/if}
			
			<div style="display:block;margin-top:10px;margin-bottom:2px;"></div>
			<label for="sel1" class="form-label" style="text-align:left;font-size:12dp">矿工费:</label>

			<select title="Select" class="form-select selectpicker" bind:value={m_Select_Cost_Index} >
				{#if m_HaveCostInfo}
					<option value="1">{m_Cost_Low}(低)</option>
					<option value="2">{m_Cost_Middle}(中)</option>
					<option value="3">{m_Cost_High}(高)</option>
				{:else}
					<option value="0">正在获取...</option>
				{/if}
			</select>

			<button class="btn btn-primary btn-block" on:click={OnSubmit}>
				<span class="spinner-border-sm"></span>
				{#if g_isSubmit}
				<span class="spinner-border spinner-border-sm"></span>
				正在提款..
				{:else}
				测试提款
				{/if}
			</button>

			{#if m_ErrorText}
				<label class="form-label" style="text-align:left;font-size:12dp;color:red">{m_ErrorText}</label>
			{/if}
		</div>
	</div>
</main>

<style>
	main {
		text-align: center;
		padding: 0;
		margin: 0 auto;
	}
</style>