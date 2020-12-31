# COD 2
- TU5 ID: 0x415607D1

## Structs
```cpp

struct Material
{
	const char* name;
};

struct Font
{
	const char* name;
	int pixelHeight;
};

struct UiContext
{
	char pad_0000[24]; //0x0000
	int32_t scrWidth; //0x0018
	int32_t scrHeight; //0x001C
	char pad_0020[4]; //0x0020
	float fps; //0x0024
	char pad_0028[24]; //0x0028
};

struct refdef_t
{
	char pad_0000[8]; //0x0000
	int32_t scrWidth; //0x0008
	int32_t scrHeight; //0x000C
	float tanfX; //0x0010
	float tanfY; //0x0014
	vec3_t viewOrigin; //0x0018
	vec3_t viewAxis[3]; //0x0024
	char pad_0048[16]; //0x0048
	vec3_t viewAxis2[3]; //0x0058
	char pad_007C[20]; //0x007C
	float tanFoV; //0x0090

};

struct centity_t
{
	int8_t eType; //0x0000
	char pad_0001[31]; //0x0001
	vec3_t origin; //0x0020
	char pad_002C[344]; //0x002C
};

enum XAsset
{
	XANIM,
	XMODEL,
	MATERIAL,
	IMAGE,
	SOUND,
	SNDCURVE,
	COL_MAP_SP,
	COL_MAP_MP,
	GFX_MAP,
	LIGHTDEF,
	UI_MAP,
	FONT,
	MENU,
	LOCALIZE,
	WEAPON,
	SNDDRIVERGLOBALS,
	IMPACTFX,
	AITYPE,
	MPTYPE,
	CHARACTER,
	XMODELALIAS,
	RAWFILE,
};


UiContext* cgDC = (UiContext*)0x84004C20;
refdef_t* refdef = (refdef_t*)0x8272F1B8;
```

## Hook
```cpp
Menu_PaintAll(int r3, int r4) - 0x822FCA58
R_RenderScene(refdef_t *refdef, int frameTime, int *viewParmsDraw, int *viewParmsDpvs) - 0x8237CDE8
CG_EntityEvent(centity_t* cen, int event) - 0x82451D38
DWORD XamInputGetState_Hook(int userIndex, int flags, PXINPUT_STATE pState) - 824956D4 

```

## Functions
```cpp
typedef int* (*DB_FindXAssetHeader_t)(XAsset index, const char* name, int r5) - 0x82401CE0
typedef void (*UI_DrawText_t)(const char* text, int maxChars, Font* font, float x, float y, int horzAlign, int vertAlign, float scale, float* color, int style, int localClientNum) - 0x822FE468
typedef void(*CG_DrawRotatedPicPhyiscal_t)(float x, float y, float w, float h, float angle, float* color, Material* material) - 0x82457368

typedef void(*Cbuf_AddText_t)(const char* cmd) - 0x8233EF10 
typedef void(*SV_GameSendServerCommand_t)(int null, const char* string) - 0x8230A8F0

Font* R_RegisterFont(const char* font)
{
	return (Font*)DB_FindXAssetHeader(FONT, font, 0);
}

Material* Material_RegisterHandle(const char* shader)
{
	return (Material*)DB_FindXAssetHeader(MATERIAL, shader, 0);
}

void iPrintln(const char* string)
{
	SV_GameSendServerCommand(0, va("%s \"%s\"", "gm", string));
}

void iPrintlnBold(const char* string)
{
	SV_GameSendServerCommand(0, va("%s \"%s\"", "gmb", string));
}

```

## WorldToScreen
```cpp
bool WorldToScreen(vec3_t point, vec2_t* world)
{
	vec3_t trans;
	vec_t xc, yc;
	vec_t px, py;
	vec_t z;

	px = tan(refdef->tanfX * M_PI / 360.0);
	py = tan(refdef->tanfY * M_PI / 360.0);

	trans = point - refdef->viewOrigin;

	xc = refdef->scrWidth / 2.0;
	yc = refdef->scrHeight / 2.0;

	z = trans.DotProduct(refdef->viewAxis[0]);
	if (z <= 0.001)
		return false;

	world->x = xc - trans.DotProduct( refdef->viewAxis[1]) * xc / (z * px);
	world->y = yc - trans.DotProduct(refdef->viewAxis[2]) * yc / (z * py);
	return true;
}
```
