# app-script-api

# code

```javascript
const ss = SpreadsheetApp.openByUrl("https://docs.google.com/spreadsheets/.......")  // google sheet url

// need to remove the skip and limit from url
// the pass it to the data
// Need to make an another function to limit the data

const sheet = ss.getSheetByName(".....")  // sheet name
function doGet(e) {
  let data = sheet.getDataRange().getValues()
  let obj = {
    total: data.length,
    length: '',
    skip: e?.parameter?.skip,
    limit: e?.parameter?.limit,
    search: e?.parameter?.search || '',
    paidOrFree: e?.parameter?.paidOrFree || '',
    category: e?.parameter?.category || '',
    header: data[0],
    data: [],
  };
  data = getJsonArrayFromData(data)
  obj.data = searchData(data, obj)
  obj.total = obj.data.length
  obj.data = paginatedTheData(obj.data, e?.parameter?.skip, e?.parameter?.limit)
  obj.data = obj.data.map(value => {
    return {
      ...value,
      categories: strToArray(value.categories)
    }
  })
  
  // obj.data=await Promise.all(obj.data.map(async(value)=>{
  //   const image=await getImageToUrl(value.link)
  //   return {
  //     ...value,
  //     logos:image
  //   }
  // }))
  
  obj.length = obj.data.length

  // Logger.log(data)
  return ContentService.createTextOutput(JSON.stringify(obj)).setMimeType(ContentService.MimeType.JSON)
}

/**
 * This function will limit the data by given params
 * @params data   Collections for website it could anything
 * @params skip   Data we need to skip
 * @params limit  Limited data we need to send to the server
 */

function paginatedTheData(data, skip = 0, limit = 20) {
  try {
    if (data.length == 0 || limit == 0) {
      return data;
    }
    let filterResult = data.splice(skip, limit)
    // Logger.log(data.length)
    // Logger.log(filterResult.length)
    // Logger.log(filterResult)
    return filterResult;
  } catch (error) {
    Logger.error(error.message)
    return { error: error.message }
  }
}

function searchData(data, searches) {
  let { search, paidOrFree, category } = searches
  let filteredData = data
  if (search) {
    filteredData = filteredData.filter(value => {
      const { title, description, paid_or_free, link, brief_and_its_features, categories, reviews, rating_score, logos } = value
      return new RegExp(search, "gi").test(title) || new RegExp(search, "gi").test(description) || new RegExp(search, "gi").test(categories) || new RegExp(search, "gi").test(brief_and_its_features)
    })
  }
  if (paidOrFree) {
    filteredData = filteredData.filter(value => {
      const { title, description, paid_or_free, link, brief_and_its_features, categories, reviews, rating_score, logos } = value
      return new RegExp(paidOrFree, "gi").test(paid_or_free)
    })
  }
  if (category) {
    filteredData = filteredData.filter(value => {
      const { title, description, paid_or_free, link, brief_and_its_features, categories, reviews, rating_score, logos } = value
      return new RegExp(category, "gi").test(categories)
    })
  }
  return filteredData
}

function getJsonArrayFromData(data) {
  let obj = {};
  let result = [];
  let headers = data[0];
  let cols = headers.length;
  let row = [];
  for (let i = 1, l = data.length; i < l; i++) {
    // get a row to fill the object
    row = data[i];
    // clear object
    obj = {
      id:i-1
    };
    for (let col = 0; col < cols; col++) {
      // fill object with new values
      obj[headers[col].trim().toLowerCase().replace(/ /g, '_')] = row[col];
    }
    // add object in a final result
    result.push(obj);
  }
  return result;
}


function strToArray(str) {
  let result = str?.split("*")?.join(",");
  return result?.split(",")?.filter((f) => !!f)?.map(v => v?.trim()) || []
}

// const urlToImageApiUrl = 'https://ss.thinkgestalt.net.in/api/images/website-screenshot'
// async function getImageToUrl ( link ) {
//   let data = await fetch( `${urlToImageApiUrl}?url=${link}` );
//   data = await data.json();
//   return data
// }
```
