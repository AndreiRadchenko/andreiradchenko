# Creating components for services pages and adding its to the pages.

Every component instance adds `ALL` its data to the services page model, submit its data to the page document in DB and fetch data in the same query as a parent service page. This way, we get separated component instance on the each page, that could be managed independently from others.

## Backend component implementation:

1. Create component interface in `src/interfaces/services/ServicesComponents.interface.ts`

```ts
export interface IFreeServicesComponent {
  title: string;
  services: IFreeService[];
}
```

2. Add component data to the page data interface. For example `src/interfaces/services/cloud.interface.ts`

```ts
export interface ICloud {
    headerBlock: {
        title: string;
        text: string;
        button: string;
        buttonLink: string;
        image: { url: string };
    };
    ...
    freeServices?: IFreeServicesComponent;
    meta: IMeta;
}
```

3. Create component mongoose schema in the folder `src/models/services/ServicesComponent`

```ts
export const FreeServicesSchema = new Schema(
  {
    title: String,
    services: [FreeService],
  },
  { timestamps: false, _id: false }
);
```

4. Add component schema to the service page schema. For example `src/models/services/Cloud.ts`

```ts
import { FreeServicesSchema } from "./ServicesComponent";

const CloudSchema = new Schema({
  headerBlock: {},
  worthBlock: {},
  providesBlock: {},
  projects: [String],
  workBlock: {},
  footerBlock: {},
  freeServices: FreeServicesSchema,
  meta: metaTagsSchema,
});

export const Cloud = model("cloud", CloudSchema);
```

5. Add component data template to the page document in DB.

```json
  "freeServices": {
    "title": "Bonuses you’ll receive as a gift",
    "services": [
      {
        "subtitle": "Maintenance",
        "text": "Get a reliable team for the long run which takes care of the project for 6 months after the release for free."
      },
      ...
      {
        "subtitle": "Code review",
        "text": "Your code will surpass industry standards via rigorous and free code reviews."
      }
    ]
  }
```

## Frontend component implementation:

1. Create component interface in `src/types/ServicesComponent.types.ts`

```ts
export interface IFreeServicesComponent {
  title: string;
  services: IFreeService[];
}
```

2. Add component data to the page data interface in the `src/types/Admin/Response.types.ts`

```ts
export interface ICloudService {
  meta: IMetaBlock;
  headerBlock: {
  };
  ...
  freeServices?: IFreeServicesComponent;
}
```

3. Create component folder in the folder `src/components/ServisesComponents`. Create subfolder for admin page component and for service page component.

4. Implement React components for admin page adn for admin page.

5. Insert newly created admin component into the admin page of the service:

```ts
<AdminBlockDropDown title="FREE SERVICES">
  <FreeServices />
</AdminBlockDropDown>
```

6. Insert newly created services component into the page of the service:

```ts
<Layout>
  <Styled.Layout>
    <WorkBlock />
    <FreeService freeServices={data?.freeServices} />
    <FooterBlock />
  </Styled.Layout>
</Layout>
```

# Adding components to the pages.

## Backend:

1. Add component data to the page data interface. For example `src/interfaces/services/cloud.interface.ts`

```ts
export interface ICloud {
    headerBlock: {
        title: string;
        text: string;
        button: string;
        buttonLink: string;
        image: { url: string };
    };
    ...
    freeServices?: IFreeServicesComponent;
    meta: IMeta;
}
```

2. Add component schema to the service page schema. For example `src/models/services/Cloud.ts`

```ts
import { FreeServicesSchema } from "./ServicesComponent";

const CloudSchema = new Schema({
  headerBlock: {},
  worthBlock: {},
  providesBlock: {},
  projects: [String],
  workBlock: {},
  footerBlock: {},
  freeServices: FreeServicesSchema,
  meta: metaTagsSchema,
});

export const Cloud = model("cloud", CloudSchema);
```

3. Add component data template to the page document in DB.

```json
  "freeServices": {
    "title": "Bonuses you’ll receive as a gift",
    "services": [
      {
        "subtitle": "Maintenance",
        "text": "Get a reliable team for the long run which takes care of the project for 6 months after the release for free."
      },
      ...
      {
        "subtitle": "Code review",
        "text": "Your code will surpass industry standards via rigorous and free code reviews."
      }
    ]
  }
```

## Frontend:

1. Add component data to the page data interface in the `src/types/Admin/Response.types.ts`

```ts
export interface ICloudService {
  meta: IMetaBlock;
  headerBlock: {
  };
  ...
  freeServices?: IFreeServicesComponent;
}
```

2. Insert admin component into the admin page of the service:

```ts
<AdminBlockDropDown title="FREE SERVICES">
  <FreeServices />
</AdminBlockDropDown>
```

3. Insert services component into the page of the service:

```ts
<Layout>
  <Styled.Layout>
    <WorkBlock />
    <FreeService freeServices={data?.freeServices} />
    <FooterBlock />
  </Styled.Layout>
</Layout>
```
