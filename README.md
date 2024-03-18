# Money-Tracking-Using-Telegram-Bot
#CODE
// Constants

const TOKEN = `6347352422:AAF_3vG11XVj_UpX6LQyzvcZbs4B8RKElys`;
const BASE_URL = `https://api.telegram.org/bot${TOKEN}`;
const CHAT_ID = '1291643009';
const DEPLOYED_URL = 'https://script.google.com/macros/s/AKfycbzqydRrxmHs9iVaMSWDTw1j3IXLQ3HhySh77KRxIYoNffkL0dCBzy8BMi0-1hhz4gTN/exec';
const SUM_CELL = 'E2';
const SUM_CELL_CASH = 'F2';
const SUM_CELL_BANK = 'G2';
const METHODS = {
  SEND_MESSAGE: 'sendMessage',
  SET_WEBHOOK: 'setWebhook',
  GET_UPDATES: 'getUpdates',
}

// Utils

const toQueryParamsString = (obj) => {
  return Object.keys(obj)
    .map(key => `${encodeURIComponent(key)}=${encodeURIComponent(obj[key])}`)
    .join('&');
}

// Telegram APIs

const makeRequest = async (method, queryParams = {}) => {
  const url = `${BASE_URL}/${method}?${toQueryParamsString(queryParams)}`
  const response = await UrlFetchApp.fetch(url);
  return response.getContentText();
}

const sendMessage = (text) => {
  makeRequest(METHODS.SEND_MESSAGE, {
    chat_id: CHAT_ID,
    text
  })
}

const setWebhook = () => {
  makeRequest(METHODS.SET_WEBHOOK,{
    url: DEPLOYED_URL
  })
}

const getChatId = async () => {
  const res = await makeRequest(METHODS.GET_UPDATES);
  console.log("ChatId: ", JSON.parse(res)?.result[0]?.message?.chat?.id)
}

// Google Sheet

const addNewRow = (content = []) => {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const Avals = sheet.getRange("A1:A").getValues();
  const Alast = Avals.filter(String).length;
  const columnNumber = content.length;
  const newRow = sheet.getRange(Alast + 1, 1, 1, columnNumber);
  newRow.setValues([content]);
}

// Extract label & price

const getMultiplyBase = (unitLabel) => {
  switch (unitLabel) {
    case 'k':
    case 'K':
    case 'nghìn':
    case 'ng':
    case 'ngàn':
      return 1000;
    case 'lít':
    case 'lit':
    case 'l':
      return 100000;
    case 'củ':
    case 'tr':
    case 'm':
    case 'M':
      return 1000000;
    default:
      return 1;
  }
};

const addExpense = (text) => {
  const regex = /(.*)\s(\d+)(\w+)\s(\w+)/g; // Regex để phù hợp với cú pháp "an trua 20k cash"
  const label = text.replace(regex, '$1 $2'); // Lấy tên của món ăn (Expense)
  const priceText = text.replace(regex, '$2'); // Lấy giá của món ăn (Price)
  const unitLabel = text.replace(regex, '$3'); // Lấy đơn vị tiền tệ (Denomination)
  const paymentMethod = text.replace(regex, '$4'); // Lấy phương thức thanh toán (Payment Method)
  const time = new Date().toLocaleString();
  const price = Number(priceText) * getMultiplyBase(unitLabel);

  addNewRow([time, label, price, paymentMethod]); // Thêm một hàng mới vào Google Sheet với thông tin của chi tiêu (Add a new row in google sheet for each element)
}


// Webhooks

const doPost = (request) =>{
  const contents = JSON.parse(request.postData.contents);
  const text = contents.message.text;
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  addExpense(text); 

  // Tính tổng chi tiêu tổng cộng (Sum of all)
  const totalExpenses = sheet.getRange(SUM_CELL).getValue().toLocaleString('vi-VN', {style : 'currency', currency : 'VND'});

  // Tính tổng chi tiêu cho từng phương thức thanh toán (Sum of Payment Method: Cash or Bank)
  const range = sheet.getRange('C:D');
  const values = range.getValues();
  let totalCash = 0;
  let totalBank = 0;

  console.log(values)

  for (let i = 0; i < values.length; i++) {
    if (values[i][1] === 'Cash') {
      totalCash += values[i][0];
    } else if (values[i][1] === 'Bank') {
      totalBank += values[i][0];
    }
  }
  
  // Gửi tin nhắn trả lời (Send Reply Message To Telegram)
  sendMessage(`Tổng chi tiêu: ${totalExpenses}\nTổng chi = Cash: ${totalCash.toLocaleString('vi-VN', {style : 'currency', currency : 'VND'})}\nTổng chi = Bank: ${totalBank.toLocaleString('vi-VN', {style : 'currency', currency : 'VND'})}`);
}


#Instruction


First of all, create a new telegram bot using BotFather --> /newbot for create bot.
# How can I use this code?
Secondly, create a new google sheet --> Extension --> Add my code --> Switch to setWebhook and Run --> 
![image](https://github.com/datheobc123/Money-Tracking-Using-Telegram-Bot/assets/142462660/ac1e54b1-3501-4791-af2e-d54567567f73)



Deploy --> Run again.
![image](https://github.com/datheobc123/Money-Tracking-Using-Telegram-Bot/assets/142462660/c05fe959-a489-468a-8d5c-73e2ee0dd01f)



Copy Script and add right here and run again:

![image](https://github.com/datheobc123/Money-Tracking-Using-Telegram-Bot/assets/142462660/861540e9-4a89-4e33-98da-591374071f3a)





Done! Congratulations.
Now you can convert data from telegram to google sheet
For example: Have lunch 15k Cash
Have lunch (Name of your expense) / 15k (15k = 15.000 Vietnam Dong) / Cash (Payment Method)
So, in google sheet it will be like this:

![image](https://github.com/datheobc123/Money-Tracking-Using-Telegram-Bot/assets/142462660/50fb7de5-cd46-428f-b812-1168ca5b842c)

You can use a new column to calculate sum of each method or sum of all by using =Sum() or =Sumifs()
