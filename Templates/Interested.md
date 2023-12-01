<%*
const dv = app.plugins.plugins["dataview"].api;
var content = tp.file.content;
let dvPage = dv.page(tp.file.title);
const urls = {
  ddc: "http://classify.oclc.org/classify2/ClassifyDemo?search-title-txt=",
  kindle: "https://www.amazon.com/s?k=",
}

stageInterested();

async function stageInterested() {
  // STATUS
  await replaceField("status", "interested");

  // Add missing data
  if (!dvPage.id) {
    await fieldsFromApi();
  }

  // Write content
  const tFile = tp.file.find_tfile(tp.file.title);
  const newContent = content;
  await app.vault.modify(tFile, newContent);
}

async function fieldsFromApi() {
  const fields = "fields=items/volumeInfo(authors,pageCount,publishedDate)";
  // Variables
  let bookQuery = await tp.system.prompt("Enter book");
  const url = `https://www.googleapis.com/books/v1/volumes/${dvPage.id}?${fields}`;

  try {
    const response = await requestUrl({
      url: url
    })
    let bookObj = response.json;
    let volInfo = bookObj.volumeInfo;
    await replaceField("Author", `[[${volInfo.authors[0]}]]`);
    await replaceField("pages", volInfo.pageCount);
    await replaceField("Year published", `[[${volInfo.publishedDate.slice(0, 4)}]]`)
  } catch (error) {
    new Notice("⚠️ " + error);
  }
}

async function replaceFromUrl(link, field) {
  let url = link + encodeURI(tp.file.title);
  window.open(url);
  let info = await tp.system.prompt(field, "", "") || "none";
  await replaceField(field, info);
}

async function replaceField(field, newValue) {
  const find = new RegExp(`\\[${field}:: .*?\\]`, 'g');
  console.log(find);
  const replace = `[${field}:: ${newValue}]`

  content = content.replace(find, replace);
}
%>