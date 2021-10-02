# TEST
enum RepGB
{
    Reptex[120],
    RepTime,
};
new RepInfo[100][RepGB];


CMD:report(playerid,params[])
{
    if(PLAYER_DATA[playerid][DATA_LOGIN] == false) return SendClientMessage(playerid,-1,"Вы не авторизованы на сервере!");
    ReportDialog(playerid);
	return true;
}

CMD:reports(playerid)
{
	if(PLAYER_DATA[playerid][data_ADMLVL] == 0) return SCM(playerid, -1, "Команда доступна только для администрации!");
	new str[200], string[1500], null = 0;
	foreach(new i:Player)
	{
	    if(GetPVarInt(i, "RepUn") > 0)
	    {
	        null++;
            format(str,sizeof(str),"%s\n{FFFFFF}%i. %s[%d] | Жалоба: %s [%s]",string, null,PLAYER_DATA[i][data_NAME],i,RepInfo[i][Reptex], date("%hh:%ii:%ss", RepInfo[i][RepTime]));
            strcat(string,str);
        }
    }
    strcat(string, "\n\n{FFCC00}Ответить на жалобу: /pm [id] [ответ]");
    if(null == 0) return SendClientMessage(playerid, 0xAA3333AA,"Список репортов пуст");
    return ShowPlayerDialogFix(playerid, 9692, 0, "Репорт",string, "Обновить", "Закрыть");
}

stock ReportDialog(playerid)
{
	new dtext[700];
	strcat(dtext, "{FFFFFF}Вы собираетесь написать Администрации сервера\n");
	strcat(dtext, "{FFFFFF}Перед тем как отправить сообщение\n");
	strcat(dtext, "{FFFFFF}убедитесь, что один из пунктов помощи не дал Вам ответа на Ваш вопрос\n\n");
	strcat(dtext, "{FF3300}Запрещено:\n");
	strcat(dtext, "{FFFFFF}- флуд, сквернословие, оффтоп\n");
	strcat(dtext, "{FFFFFF}- Выпрашивание игровых ценностей ('дать денег', 'дать лидерку', 'дать права')\n");
	strcat(dtext, "{FFFFFF}- ложные сообщения о нарушении\n\n");
	strcat(dtext, "{FF3300}За нарушение правил Администратор может:\n");
	strcat(dtext, "{FFFFFF}- предупредить (warn)\n");
	strcat(dtext, "{FFFFFF}- отключить от сервера (kick)\n");
	strcat(dtext, "{FFFFFF}- лишить возможности писать (mute)\n");
	strcat(dtext, "{FFFFFF}- заблокировать (ban)\n\n");
	strcat(dtext, "{FFFFFF}Данные правила установлены для всех игроков {339966}rglrp.tk");
	ShowPlayerDialogFix(playerid,dialog_REPORT,DIALOG_STYLE_INPUT,"{FFCC00}Репорт",dtext,"Отправить","Назад");
	return true;
}

case dialog_REPORT:
{
    if(!response) return true;
    if(response)
    {
        if(strlen(inputtext) < 1 || strlen(inputtext) > 100) return SendClientMessage(playerid,COLOR_WARNING,"Не менее 1 и не более 100 символов!"),ReportDialog(playerid);
        if(GetPVarInt(playerid,"RepUn") > 0) return SendClientMessage(playerid,0xAA3333AA,"Ошибка: Ваша прошлая жалоба ещё не рассмотрена");
        SetPVarInt(playerid,"RepUn",1);
        strmid(RepInfo[playerid][Reptex], inputtext,0,strlen(inputtext),130);
      	RepInfo[playerid][RepTime] = gettime(); 
        SendAdminMessage(0x33FF66FF, "{FFCC00}Поступила новая жалоба от игрока! Рассмотреть жалобы: /reports");
        SendClientMessage(playerid,0x3399feFF, "Ваша жалоба отправлена на рассмотрение!");
    }
}

CMD:pm(playerid,params[])
{
    if(PLAYER_DATA[playerid][data_ADMLVL] < 1) return true;
	if(sscanf(params,"us[100]",params[0],params[1])) return SendClientMessage(playerid,COLOR_WARNING,"Используйте: /pm [ид] [текст]");
	if(!IsPlayerConnected(params[0]))return  SendClientMessage(playerid,COLOR_WARNING,"Данного ID нет на сервере!");
	new string[300];
    format(string, sizeof(string), "[Ответ] %s[%d] игроку %s[%d]: {ffffff}%s", PLAYER_DATA[playerid][data_NAME],playerid,PLAYER_DATA[params[0]][data_NAME],params[0],params[1]);
    SendAdminMessage(0xffa141FF, string);
	format(string, sizeof(string), "Администратор %s[%d] ответил вам: {ffffff}%s", PLAYER_DATA[playerid][data_NAME],playerid,params[1]);
    SendClientMessage(params[0], 0xffa141FF, string);
    if(strlen(RepInfo[params[0]][Reptex]))
    {
    	format(string, sizeof(string), "На вашу жалобу: {FFFFFF}%s", RepInfo[params[0]][Reptex]);
    	SendClientMessage(params[0], 0xffa141FF, string);
	}
	strmid(RepInfo[playerid][Reptex], "",0,0,0);
    DeletePVar(params[0], "RepUn");
    return true;
}
