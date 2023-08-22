# Project cgsteam.io Adding history of changes to the admin panel section.

<img src="public/HistoryOfChanges/Screenshot from 2023-08-22 14-30-25.png" width="45%"></img>
|
<img src="public/HistoryOfChanges/Screenshot from 2023-08-22 14-31-28.png" width="45%"></img>

Page models i.e., `Contact`, `Web` (web services) etc that implement history of
changes create its own collection in db. All history collection are placed in
`history` db and have name like `contacts_history`, `webs_history`. This
collections will be created automatically after passing all steps below and
making changes to the page in the admin panel.

## Backend implementation of page history:

1. Install dependency: `npm i mongoose-history`

<div id="backend2"></div>

2. Add `lastModified` field to the section of page model (to the first nesting
   models fields ) `src/models/services/Web.ts`

```ts
const WebSchema = new Schema({
    headerBlock: {
        title: String,
        text: String,
        button: String,
        buttonLink: String,
        image: { url: String },
        lastModified: String,
    },
    ...
    solutionBlock: {
        subtitle: String,
        text: String,
        lastModified: String,
    },
```

3. Import custom mongoose plugin and HistoryDbConnection to the schema module,
   and create history model for this page: `src/models/services/Web.ts`

```ts
import mongooseHistory from '../history/mongoose-history/lib/mongoose-history';
import HistoryDbConnection from '../../repository/history/historyConnection';

const WebSchema = new Schema({

    ...

    WebSchema.plugin(mongooseHistory as any, {
    historyConnection: HistoryDbConnection,
    // Time to live in seconds (60 days)
    ttl: 5184000,
});

export const Web = model('web', WebSchema);
export const Webs_history = (Web as any).historyModel();
}
```

4. Add `timeStampMiddleware` to the page router. Pass page service as argument
   to this middleware. It adds `lastModified` field on the `req.body` and adds
   `req.pageData` that content "old" page data.

   `src/routes/services/webRouter.ts`

```ts
import { Router } from 'express';
import { webController } from '../../controllers/services/webController';
import { timeStampMiddleware, tokenMiddleware } from '../../middlewares/';
import { webRepository } from '../../repository/services/webRepository';

const router = Router();

router.get('/', webController.getAll);
router.put(
  '/',
  tokenMiddleware.checkAccessToken,
  timeStampMiddleware(webRepository),
  webController.update,
);

export const webRouter = router;
```

5. Import `IRequestPageData` interface to the page controller module and made
   some changes to the `update` method:
   `src/controllers/services/webController.ts`

```ts
import { IRequestPageData } from '../../interfaces';

class WebController {

...

public async update(
        // req: Request,
        req: IRequestPageData,
        res: Response,
        next: NextFunction
    ): Promise<void> {
        try {
            // const updatedData = req.body;
            // const page = await webRepository.getAll();
            const pageData = req.pageData;

            // if (page?.length) {
            if (pageData?.length) {
                // await webRepository.update({ ...updatedData });
                await webRepository.update({ ...req.body });
                res.json({
                    message: 'Updated',
                });
                return;
            }
            // await webRepository.update({ ...updatedData });
            await webRepository.create({ ...req.body });
            res.json({
                message: 'Created',
            });
        } catch (e) {
            next(e);
        }
    }
}
```

<div id="backend6"></div>

6. Import early created page history model to the:
   `src/repository/history/historyRepository.ts`. Add it as field to the
   `historyRepositories` object. You should use page name from query param
   string as field key. For example `web: Webs_history,` for the web page query
   http://localhost:3000/history/web/headerBlock

```ts
import { Contacts_history } from '../../models/contact/Contact';
import { Webs_history } from '../../models/services/Web';

interface IHistoryRepository {
  [key: string]: any;
}

const historyRepositories: IHistoryRepository = {
  contacts: Contacts_history,
  web: Webs_history,
};

class HistoryRepository {
  public async getAll(page: string) {
    return historyRepositories[page].find({}).sort({ _id: -1 });
  }
  public async getSectionHistory(page: string) {
    return historyRepositories[page].find({}).sort({ _id: -1 });
  }
}

export const historyRepository = new HistoryRepository();
```

## Frontend history component implementation:

1. Add `lastModified` fields to the page interface. Insert it to the section
   block like in [Backend Schema ](#backend2) (to the first nesting fields )

   `src/types/Admin/Response.types.ts`

```ts
export interface IServiceWeb {
  headerBlock: {
    title: string;
    text: string;
    button: string;
    buttonLink: string;
    image: { url: string };
    lastModified?: string;
  };
  comparisonBlock: {
    desktopColumn: ISubtitleWithList;
    webColumn: ISubtitleWithList;
    lastModified?: string;
  };
  ...
}
```

2. To update time in `<HistoryLink />` component add next imports and code to
   the section component module

   `src/components/Admin/Services/Web/AdminHeadBlockWeb.tsx`

```ts
import { useQueryClient } from '@tanstack/react-query';
import HistoryLink from '../../HistoryLink';
import { queryKeys } from '../../../../consts/queryKeys';

...

const AdminHeadBlockWeb = () => {
  const queryClient = useQueryClient();
  const data = queryClient.getQueryData<IServiceWeb>([
    queryKeys.getServiceWebPage,
  ])?.headerBlock;
```

3. Add markup above button for presenting last modification time and link to the
   history of changes

   `src/components/Admin/Services/Web/AdminHeadBlockWeb.tsx`

```ts
   ...
   </AdminHeaderGrid>
      {data?.lastModified && (
        <HistoryLink
          sectionName="Head Block"
          lastModified={data?.lastModified}
          link={"/history/web/headerBlock"}
        />
      )}
      <BlackButton
   ...
```

Props `Link` is a history query rout. Where `'web'` is a
[key of history model](#backend6) and `'headerBlock'` is name of
[Schema section](#backend2).

\*\*Caution! Some section might use render logic sensible to the new
`lastModified` field. Like this: `src/components/WebService/WebPros.tsx`

```ts
  comparisonBlock: {
    desktopColumn: ISubtitleWithList;
    webColumn: ISubtitleWithList;
    lastModified?: string;
  };
   ...
   {data &&
    Object.values(data).map(
      (category, idx) =>
        typeof category !== "string" && (
            <div key={`${category} ${idx}`}>
            <Styled.BgMobileImage
                    src={bgMobileImage.src}
                    alt="mobile image"
                  />
   ...
```

Before modification, `data` had contained two fields. I had had to add type
check to the render logic to prevent page crashing with new `lastModified`
field.

## Modify reusable component:

1. Add import to the component module
   `src/components/ServisesComponents/FreeServices/AdminComponent/FreeServices.tsx`

```ts
import { useQueryClient } from '@tanstack/react-query';
import { IServiceHistory } from '../../../../types/ServicesComponent.types';
```

2. Add props `serviceName` and `queryKey` to the component. Get data form
   `queryClient`
   `src/components/ServisesComponents/FreeServices/AdminComponent/FreeServices.tsx`

```ts
...
const FreeService = <T extends IFreeService>({
  serviceName = "",
  queryKey = "",
}: IServiceHistory) => {
  const queryClient = useQueryClient();
  const data = queryClient.getQueryData<T>([queryKey])?.freeServices;
...
```

3. Add markup above button for presenting last modification time and link to the
   history of changes.
   `src/components/ServisesComponents/FreeServices/AdminComponent/FreeServices.tsx`

```ts
...
 </FieldArray>
      <div>
        {data?.lastModified && (
          <HistoryLink
            sectionName="Free services"
            lastModified={data?.lastModified}
            link={`/history/${serviceName}/freeServices`}
          />
        )}
        <BlackButton
...
```

## Customize reusable component on service page:

1. Add import to the service page module:
   `src/components/Admin/Services/Web/index.tsx`

```ts
import { queryKeys } from '../../../../consts/queryKeys';
```

2. Add history props to the reusable component:
   `src/components/Admin/Services/Web/index.tsx`

```ts
<FreeServices serviceName={'web'} queryKey={queryKeys.getServiceWebPage} />
```

`serviceName` string is used as a page name param for query. For example for the
history query of freeServices on the web service page
http://localhost:3000/history/web/freeServices. It should be the same as a
[key of history model](#backend6) in
`src/repository/history/historyRepository.ts`

```ts
const historyRepositories: IHistoryRepository = {
  contacts: Contacts_history,
  web: Webs_history,
};
```
