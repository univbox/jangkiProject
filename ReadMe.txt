========================================================================
       MICROSOFT FOUNDATION CLASS LIBRARY : Jangki
========================================================================


AppWizard has created this Jangki application for you.  This application
not only demonstrates the basics of using the Microsoft Foundation classes
but is also a starting point for writing your application.

This file contains a summary of what you will find in each of the files that
make up your Jangki application.

Jangki.dsp
    This file (the project file) contains information at the project level and
    is used to build a single project or subproject. Other users can share the
    project (.dsp) file, but they should export the makefiles locally.

Jangki.h
    This is the main header file for the application.  It includes other
    project specific headers (including Resource.h) and declares the
    CJangkiApp application class.

Jangki.cpp
    This is the main application source file that contains the application
    class CJangkiApp.

Jangki.rc
    This is a listing of all of the Microsoft Windows resources that the
    program uses.  It includes the icons, bitmaps, and cursors that are stored
    in the RES subdirectory.  This file can be directly edited in Microsoft
	Visual C++.

Jangki.clw
    This file contains information used by ClassWizard to edit existing
    classes or add new classes.  ClassWizard also uses this file to store
    information needed to create and edit message maps and dialog data
    maps and to create prototype member functions.

res\Jangki.ico
    This is an icon file, which is used as the application's icon.  This
    icon is included by the main resource file Jangki.rc.

res\Jangki.rc2
    This file contains resources that are not edited by Microsoft 
	Visual C++.  You should place all resources not editable by
	the resource editor in this file.



/////////////////////////////////////////////////////////////////////////////

For the main frame window:

MainFrm.h, MainFrm.cpp
    These files contain the frame class CMainFrame, which is derived from
    CFrameWnd and controls all SDI frame features.

res\Toolbar.bmp
    This bitmap file is used to create tiled images for the toolbar.
    The initial toolbar and status bar are constructed in the CMainFrame
    class. Edit this toolbar bitmap using the resource editor, and
    update the IDR_MAINFRAME TOOLBAR array in Jangki.rc to add
    toolbar buttons.
/////////////////////////////////////////////////////////////////////////////

AppWizard creates one document type and one view:

JangkiDoc.h, JangkiDoc.cpp - the document
    These files contain your CJangkiDoc class.  Edit these files to
    add your special document data and to implement file saving and loading
    (via CJangkiDoc::Serialize).

JangkiView.h, JangkiView.cpp - the view of the document
    These files contain your CJangkiView class.
    CJangkiView objects are used to view CJangkiDoc objects.



/////////////////////////////////////////////////////////////////////////////
Other standard files:

StdAfx.h, StdAfx.cpp
    These files are used to build a precompiled header (PCH) file
    named Jangki.pch and a precompiled types file named StdAfx.obj.

Resource.h
    This is the standard header file, which defines new resource IDs.
    Microsoft Visual C++ reads and updates this file.

/////////////////////////////////////////////////////////////////////////////
Other notes:

AppWizard uses "TODO:" to indicate parts of the source code you
should add to or customize.

If your application uses MFC in a shared DLL, and your application is 
in a language other than the operating system's current language, you
will need to copy the corresponding localized resources MFC42XXX.DLL
from the Microsoft Visual C++ CD-ROM onto the system or system32 directory,
and rename it to be MFCLOC.DLL.  ("XXX" stands for the language abbreviation.
For example, MFC42DEU.DLL contains resources translated to German.)  If you
don't do this, some of the UI elements of your application will remain in the
language of the operating system.

/////////////////////////////////////////////////////////////////////////////
// JKStage.cpp: implementation of the CJKStage class.
//
//////////////////////////////////////////////////////////////////////

#include "stdafx.h"
#include "JKStage.h"
#include <stdlib.h>

#ifdef _DEBUG
#undef THIS_FILE
static char THIS_FILE[]=__FILE__;
#define new DEBUG_NEW
#endif

//////////////////////////////////////////////////////////////////////
// Construction/Destruction
//////////////////////////////////////////////////////////////////////

int CJKStage::sm_nCount;
// �޸���ƽ ���� �÷���
bool CJKStage::m_bPoBeforeJang;
bool CJKStage::m_bJolToCenter;
bool CJKStage::m_bCHAPOBalance;
bool CJKStage::m_bAggressivePO;
bool CJKStage::m_bAggressiveCHA;
bool CJKStage::m_bAggressiveJOL;
bool CJKStage::m_bAggressiveSANG;
bool CJKStage::m_bAggressiveMA;
bool CJKStage::m_bOnlyJangCenter;
CHistoryStack CJKStage::m_cHistoryStack;

CJKStage::CJKStage()
{
	::srand( (unsigned)time( NULL ) );
	Initialize();
}

CJKStage::CJKStage(CJKStage * pParentStage)
{
	m_pFirst = NULL;
	m_pSelectedChild = NULL;
	m_nDepth = pParentStage->m_nDepth + 1;
	m_enTeam = pParentStage->m_enTeam == TM_HAN ? TM_CHO : TM_HAN;
	m_bAmIMax = !pParentStage->IsMaxTurn();
	::memcpy(m_enPanel,pParentStage->m_enPanel,sizeof(m_enPanel));
//	GetCHAPOCount();
}

void CJKStage::GetCHAPOCount()
{
//	if ( !m_bCHAPOBalance ) return;
	m_nHANChaCount	=	0;
	m_nHANPoCount	=	0;
	m_nCHOChaCount	=	0;
	m_nCHOPoCount	=	0;
	for ( int x,y = 0; y < DF_PANEL_HEIGHT; ++ y )
		for ( x = 0; x < DF_PANEL_WIDTH; ++ x )
		{
			switch ( m_enPanel[y][x] )
			{
			case PS_HAN_CHA	: ++ m_nHANChaCount;
			case PS_HAN_PO	: ++ m_nHANPoCount;
			case PS_CHO_CHA	: ++ m_nCHOChaCount;
			case PS_CHO_PO	: ++ m_nCHOPoCount;
			}
		}
}


CJKStage::~CJKStage()
{
	ClearList();
	if ( m_pSelectedChild )
		delete m_pSelectedChild;
}

void CJKStage::Initialize()
{
	m_pFirst = NULL;
	m_pSelectedChild = NULL;
	m_cHistoryStack.ClearStack();

	::memset(m_enPanel,PS_NOTHING,sizeof(m_enPanel));
	m_enPanel[0][0] = PS_HAN_CHA;
	m_enPanel[0][1] = PS_HAN_SANG;
	m_enPanel[0][2] = PS_HAN_MA;
	m_enPanel[0][3] = PS_HAN_SA;
	m_enPanel[0][5] = PS_HAN_SA;
	m_enPanel[0][6] = PS_HAN_MA;
	m_enPanel[0][7] = PS_HAN_SANG;
	m_enPanel[0][8] = PS_HAN_CHA;
	m_enPanel[1][4] = PS_HAN_JANG;
	m_enPanel[2][1] = PS_HAN_PO;
	m_enPanel[2][7] = PS_HAN_PO;
	m_enPanel[3][0] = PS_HAN_JOL;
	m_enPanel[3][2] = PS_HAN_JOL;
	m_enPanel[3][4] = PS_HAN_JOL;
	m_enPanel[3][6] = PS_HAN_JOL;
	m_enPanel[3][8] = PS_HAN_JOL;

	m_enPanel[9][0] = PS_CHO_CHA;
	m_enPanel[9][1] = PS_CHO_MA;
	m_enPanel[9][2] = PS_CHO_SANG;
	m_enPanel[9][3] = PS_CHO_SA;
	m_enPanel[9][5] = PS_CHO_SA;
	m_enPanel[9][6] = PS_CHO_MA;
	m_enPanel[9][7] = PS_CHO_SANG;
	m_enPanel[9][8] = PS_CHO_CHA;
	m_enPanel[8][4] = PS_CHO_JANG;
	m_enPanel[7][1] = PS_CHO_PO;
	m_enPanel[7][7] = PS_CHO_PO;
	m_enPanel[6][0] = PS_CHO_JOL;
	m_enPanel[6][2] = PS_CHO_JOL;
	m_enPanel[6][4] = PS_CHO_JOL;
	m_enPanel[6][6] = PS_CHO_JOL;
	m_enPanel[6][8] = PS_CHO_JOL; 

	m_bPoBeforeJang		= true;
	m_bJolToCenter		= true;
	m_bCHAPOBalance		= true;
	m_bAggressivePO		= false;
	m_bAggressiveCHA	= true;
	m_bAggressiveJOL	= true;
	m_bAggressiveSANG	= false;
	m_bAggressiveMA		= false;
	m_bOnlyJangCenter	= true;

	GetCHAPOCount();
}

//////////////////////////////////////////////
// ������ ��ġ�� ������ ����� �� �ִ��� üũ
bool CJKStage::IsPassable(int nX,int nY)
{
	if ( nX < 0 ) return false;
	if ( nY < 0 ) return false;
	if ( nX >= DF_PANEL_WIDTH ) return false;
	if ( nY >= DF_PANEL_HEIGHT ) return false;

	if ( m_enPanel[nY][nX] == PS_NOTHING ) 
		return true;
	else
		return false;
}

void CJKStage::ClearList()
{
	if ( m_pFirst )
	{
		CJKStage *pNode = m_pFirst;
		CJKStage *pTmp;
		do
		{
			pTmp = pNode->m_pNext;
			delete pNode;
			pNode = pTmp;
		} while (pNode != NULL);
		m_pFirst = NULL;
	}
}

void CJKStage::ClearList(CJKStage *pException)
{
	if ( m_pFirst )
	{
		CJKStage *pNode = m_pFirst;
		CJKStage *pTmp;
		do
		{
			pTmp = pNode->m_pNext;
			if ( pNode != pException ) delete pNode;
			pNode = pTmp;
		} while (pNode != NULL);
		m_pFirst = NULL;
	}
}
//////////////////////////////////////////////
// ������ ��ġ�� ������ �̵��� �� �ִ��� üũ
bool CJKStage::TryToAddMovableXY(int nFromX,int nFromY,int nToX,int nToY)
{
	
	if ( nToX < 0 ) return false;
	if ( nToY < 0 ) return false;
	if ( nToX >= DF_PANEL_WIDTH ) return false;
	if ( nToY >= DF_PANEL_HEIGHT ) return false;
	
	// �̵��ϰ��� �ϴ� ���� ���� �� ���� ������ �� �����δ�.
	if ( GetTeamAt(nToX,nToY) == GetTeamAt(nFromX,nFromY) ) return false;
	
	CJKStage *pNewStage = new CJKStage(this);
	pNewStage->MovePiece(nFromX,nFromY,nToX,nToY);

	pNewStage->m_nFromX			= nFromX;
	pNewStage->m_nFromY			= nFromY;
	pNewStage->m_nToX			= nToX;
	pNewStage->m_nToY			= nToY;
	pNewStage->m_enEatenPiece	= m_enPanel[nToY][nToX];

	pNewStage->m_nHANChaCount	= m_nHANChaCount;
	pNewStage->m_nHANPoCount	= m_nHANPoCount;
	pNewStage->m_nCHOChaCount	= m_nCHOChaCount;
	pNewStage->m_nCHOPoCount	= m_nCHOPoCount;

	switch ( pNewStage->m_enEatenPiece )
	{
	case PS_HAN_CHA	: -- pNewStage->m_nHANChaCount;break;
	case PS_HAN_PO	: -- pNewStage->m_nHANPoCount;break;
	case PS_CHO_CHA	: -- pNewStage->m_nCHOChaCount;break;
	case PS_CHO_PO	: -- pNewStage->m_nCHOPoCount;break;
	} 
	int nNewScore	= pNewStage->PayOff();
	if ( sm_nCount % 1000 == 0 )
	{
		TRACE("Count - %d\n",sm_nCount);
	}

	++ sm_nCount;

	///////////////////////////////////////////////////////
	// Max���忡�� �ִ� ���� ������ ������ �����ϹǷ�
	// "��ġ�� ū �����"���� ������ �θ� ���ĺ�Ÿ ����ġ��
	// ȿ���� ����ȴ�.
	if ( m_bAmIMax )
	{
		if ( m_pFirst )
		{
			CJKStage *pNode = m_pFirst;
			do
			{
				if ( nNewScore >= pNode->GetScore() ) break;
				pNode = pNode->m_pNext;
			} while( pNode != NULL );
			if ( pNode == NULL )
			{
				m_pLast->m_pNext = pNewStage;
				pNewStage->m_pPrev = m_pLast;
				pNewStage->m_pNext = NULL;
				m_pLast = pNewStage;
			} else
			{
				CJKStage * pTmp = pNode->m_pPrev;
				pNode->m_pPrev = pNewStage;
				pNewStage->m_pNext = pNode;
				pNewStage->m_pPrev = pTmp;
				if ( pTmp )
				{
					pTmp->m_pNext = pNewStage;
				} else
				{
					m_pFirst = pNewStage;
				}
			}
		} else
		{
			m_pFirst = pNewStage;
			m_pLast = pNewStage;
			pNewStage->m_pPrev = NULL;
			pNewStage->m_pNext  = NULL;
		}
	} else
	{
		if ( m_pFirst )
		{
			CJKStage *pNode = m_pFirst;
			do
			{
				if ( nNewScore <= pNode->GetScore() ) break;
				pNode = pNode->m_pNext;
			} while(pNode != NULL);
			if ( pNode == NULL )
			{
				m_pLast->m_pNext = pNewStage;
				pNewStage->m_pPrev = m_pLast;
				pNewStage->m_pNext = NULL;
				m_pLast = pNewStage;

			} else
			{
				CJKStage * pTmp = pNode->m_pPrev;
				pNode->m_pPrev = pNewStage;
				pNewStage->m_pNext = pNode;
				pNewStage->m_pPrev = pTmp;
				if ( pTmp )
				{
					pTmp->m_pNext = pNewStage;
				} else
				{
					m_pFirst = pNewStage;
				}
			}
		} else
		{
			m_pFirst = pNewStage;
			m_pLast = pNewStage;
			pNewStage->m_pPrev = NULL;
			pNewStage->m_pNext  = NULL;
		}
	}

	return true;
}

//////////////////////////////////////////////
// ������ ��ġ�� �ִ� ���� �Ҽ� ���� ��ȯ
TEAM CJKStage::GetTeamAt(int nX,int nY)
{
	if ( m_enPanel[nY][nX] == PS_NOTHING ) return TM_NOTHING;
	if ( m_enPanel[nY][nX] < PS_BOUNDARY ) return TM_CHO;

	return TM_HAN;
}

void CJKStage::GetMovableSaAndJang(int nFromX,int nFromY)
{
	switch ( nFromY )
	{
	case 0	:
	case 7	:
		switch ( nFromX )
		{
		case 3	:
			TryToAddMovableXY(nFromX,nFromY,nFromX + 1,nFromY);
			TryToAddMovableXY(nFromX,nFromY,nFromX,nFromY + 1);
			TryToAddMovableXY(nFromX,nFromY,nFromX + 1,nFromY + 1);
			break;
		case 4	:
			TryToAddMovableXY(nFromX,nFromY,nFromX - 1,nFromY);
			TryToAddMovableXY(nFromX,nFromY,nFromX + 1,nFromY);
			TryToAddMovableXY(nFromX,nFromY,nFromX,nFromY + 1);
			break;
		case 5	:
			TryToAddMovableXY(nFromX,nFromY,nFromX - 1,nFromY);
			TryToAddMovableXY(nFromX,nFromY,nFromX,nFromY + 1);
			TryToAddMovableXY(nFromX,nFromY,nFromX - 1,nFromY + 1);
			break;
		}
		break;
	case 1	:
	case 8	:
		switch ( nFromX )
		{
		case 3	:
			TryToAddMovableXY(nFromX,nFromY,nFromX,nFromY - 1);
			TryToAddMovableXY(nFromX,nFromY,nFromX,nFromY + 1);
			TryToAddMovableXY(nFromX,nFromY,nFromX + 1,nFromY);
			break;
		case 4	:
			TryToAddMovableXY(nFromX,nFromY,nFromX - 1,nFromY);
			TryToAddMovableXY(nFromX,nFromY,nFromX - 1,nFromY - 1);
			TryToAddMovableXY(nFromX,nFromY,nFromX,nFromY - 1);
			TryToAddMovableXY(nFromX,nFromY,nFromX + 1,nFromY - 1);
			TryToAddMovableXY(nFromX,nFromY,nFromX + 1,nFromY);
			TryToAddMovableXY(nFromX,nFromY,nFromX + 1,nFromY + 1);
			TryToAddMovableXY(nFromX,nFromY,nFromX,nFromY + 1);
			TryToAddMovableXY(nFromX,nFromY,nFromX - 1,nFromY + 1);
			break;
		case 5	:
			TryToAddMovableXY(nFromX,nFromY,nFromX,nFromY - 1);
			TryToAddMovableXY(nFromX,nFromY,nFromX,nFromY + 1);
			TryToAddMovableXY(nFromX,nFromY,nFromX - 1,nFromY);
			break;
		}
		break;
	case 2	:
	case 9	:
		switch ( nFromX )
		{
		case 3	:
			TryToAddMovableXY(nFromX,nFromY,nFromX,nFromY - 1);
			TryToAddMovableXY(nFromX,nFromY,nFromX + 1,nFromY - 1);
			TryToAddMovableXY(nFromX,nFromY,nFromX + 1,nFromY);
			break;
		case 4	:
			TryToAddMovableXY(nFromX,nFromY,nFromX - 1,nFromY);
			TryToAddMovableXY(nFromX,nFromY,nFromX + 1,nFromY);
			TryToAddMovableXY(nFromX,nFromY,nFromX,nFromY - 1);
			break;
		case 5	:
			TryToAddMovableXY(nFromX,nFromY,nFromX - 1,nFromY);
			TryToAddMovableXY(nFromX,nFromY,nFromX - 1,nFromY - 1);
			TryToAddMovableXY(nFromX,nFromY,nFromX,nFromY - 1);
			break;
		}
		break;
	}
}

void CJKStage::GetMovableJol(int nFromX,int nFromY)
{
	int nDirection = GetTeamAt(nFromX,nFromY) == TM_CHO ? -1 : 1;
	
	//////////////////////////////////////////////////
	// ��,���� �����̴� ���� �ȿ��� ��,��� �Ȱ��� 
	// �밢�����ε� ������ ���ִ�.�ڷδ� ����
	switch ( nFromY )
	{
	case 1	:
	case 8	:
		switch ( nFromX )
		{
		case 4	:
			TryToAddMovableXY(nFromX,nFromY,nFromX + 1,nFromY + nDirection);
			TryToAddMovableXY(nFromX,nFromY,nFromX - 1,nFromY + nDirection);
			break;
		}
		break;
	case 2	:
	case 7	:
		switch ( nFromX )
		{
		case 3	:
			TryToAddMovableXY(nFromX,nFromY,nFromX + 1,nFromY + nDirection);
			break;
		case 5	:
			TryToAddMovableXY(nFromX,nFromY,nFromX - 1,nFromY + nDirection);
			break;
		}
		break;
	}

	// �·� ������ ����.
	TryToAddMovableXY(nFromX,nFromY,nFromX - 1,nFromY);
	// ��� ������ ����.	
	TryToAddMovableXY(nFromX,nFromY,nFromX + 1,nFromY);
	// ���� �� ����.
	TryToAddMovableXY(nFromX,nFromY,nFromX,nFromY + nDirection);	
}


void CJKStage::GetMovablePo(int nFromX,int nFromY)
{
	bool bMovable = false;
	// ���� �������� �̵����Ѻ���.
	for ( int k = nFromX + 1; k < DF_PANEL_WIDTH; ++ k )
	{
		if ( bMovable )
		{
			if ( m_enPanel[nFromY][k] != PS_CHO_PO &&
				 m_enPanel[nFromY][k] != PS_HAN_PO)
			{
				TryToAddMovableXY(nFromX,nFromY,k,nFromY);
				if ( m_enPanel[nFromY][k] != PS_NOTHING )
					break;
			} else 
			{
				break;
			}
		} else
		{
			/////////////////////////////////////////////////////
			// �� �̿��� �ٸ� ���� ���� ��� �� ����ĭ���� 
			// ���� �̵��� �� �ִ� �����̴�.
			if ( m_enPanel[nFromY][k] == PS_HAN_PO )
				break;
			if ( m_enPanel[nFromY][k] == PS_CHO_PO )
				break;
			if ( m_enPanel[nFromY][k] != PS_NOTHING )
				bMovable = true;
		}
	}
	
	bMovable = false;
	// ���� �������� �̵����Ѻ���.
	for ( k = nFromX - 1; k >= 0; -- k )
	{
		if ( bMovable )
		{
			if ( m_enPanel[nFromY][k] != PS_CHO_PO &&
				 m_enPanel[nFromY][k] != PS_HAN_PO )
			{
				TryToAddMovableXY(nFromX,nFromY,k,nFromY);
				if ( m_enPanel[nFromY][k] != PS_NOTHING )
					break;
			} else 
			{
				break;
			}
		} else
		{
			/////////////////////////////////////////////////////
			// �� �̿��� �ٸ� ���� ���� ��� �� ����ĭ���� 
			// ���� �̵��� �� �ִ� �����̴�.
			if ( m_enPanel[nFromY][k] == PS_HAN_PO )
				break;
			if ( m_enPanel[nFromY][k] == PS_CHO_PO )
				break;
			if ( m_enPanel[nFromY][k] != PS_NOTHING )
				bMovable = true;
		}
	}
	bMovable = false;
	// ���� �̵����� ����.
	for ( k = nFromY - 1; k >= 0; -- k )
	{
		if ( bMovable )
		{
			if ( m_enPanel[k][nFromX] != PS_CHO_PO &&
				 m_enPanel[k][nFromX] != PS_HAN_PO )
			{
				TryToAddMovableXY(nFromX,nFromY,nFromX,k);
				if ( m_enPanel[k][nFromX] != PS_NOTHING )
					break;
			} else 
			{
				break;
			}
		} else
		{
			/////////////////////////////////////////////////////
			// �� �̿��� �ٸ� ���� ���� ��� �� ����ĭ���� 
			// ���� �̵��� �� �ִ� �����̴�.
			if ( m_enPanel[k][nFromX] == PS_HAN_PO )
				break;
			if ( m_enPanel[k][nFromX] == PS_CHO_PO )
				break;
			if ( m_enPanel[k][nFromX] != PS_NOTHING )
				bMovable = true;
		}
	}
	bMovable = false;
	// �Ʒ��� �̵����Ѻ���.
	for ( k = nFromY + 1; k < DF_PANEL_HEIGHT; ++ k )
	{
		if ( bMovable )
		{
			if ( m_enPanel[k][nFromX] != PS_CHO_PO &&
				 m_enPanel[k][nFromX] != PS_HAN_PO )
			{
				TryToAddMovableXY(nFromX,nFromY,nFromX,k);
				if ( m_enPanel[k][nFromX] != PS_NOTHING )
					break;
			} else 
			{
				break;
			}
		} else
		{

			/////////////////////////////////////////////////////
			// �� �̿��� �ٸ� ���� ���� ��� �� ����ĭ���� 
			// ���� �̵��� �� �ִ� �����̴�.
			if ( m_enPanel[k][nFromX] == PS_HAN_PO )
				break;
			if ( m_enPanel[k][nFromX] == PS_CHO_PO )
				break;
			if ( m_enPanel[k][nFromX] != PS_NOTHING )
				bMovable = true;
		}
	}
}


void CJKStage::GetMovableCha(int nFromX,int nFromY)
{
	//////////////////////////////////////////////////
	// ��,���� �����̴� ���� �ȿ��� ��,��� �Ȱ��� 
	// �밢�����ε� ������ ���ִ�.
	switch ( nFromY )
	{
	case 0	:
	case 7	:
		switch ( nFromX )
		{
		case 3	:
			if ( GetTeamAt(4,nFromY) != GetTeamAt(nFromX,nFromY) )
				TryToAddMovableXY(nFromX,nFromY,nFromX + 1,nFromY + 1);
			if ( GetTeamAt(4,nFromY + 1) == PS_NOTHING )
				TryToAddMovableXY(nFromX,nFromY,nFromX + 2,nFromY + 2);
			break;
		case 5	:
			if ( GetTeamAt(4,nFromY) != GetTeamAt(nFromX,nFromY) )
				TryToAddMovableXY(nFromX,nFromY,nFromX - 1,nFromY + 1);
			if ( GetTeamAt(4,nFromY + 1) == PS_NOTHING )
				TryToAddMovableXY(nFromX,nFromY,nFromX - 2,nFromY + 2);
			break;
		}
		break;
	case 1	:
	case 8	:
		switch ( nFromX )
		{
		case 4	:
			TryToAddMovableXY(nFromX,nFromY,nFromX - 1,nFromY - 1);
			TryToAddMovableXY(nFromX,nFromY,nFromX + 1,nFromY - 1);
			TryToAddMovableXY(nFromX,nFromY,nFromX + 1,nFromY + 1);
			TryToAddMovableXY(nFromX,nFromY,nFromX - 1,nFromY + 1);
			break;
		}
		break;
	case 2	:
	case 9	:
		switch ( nFromX )
		{
		case 3	:
			if ( GetTeamAt(4,nFromY) != GetTeamAt(nFromX,nFromY) )
				TryToAddMovableXY(nFromX,nFromY,nFromX + 1,nFromY - 1);
			if ( GetTeamAt(4,nFromY - 1) == PS_NOTHING )
				TryToAddMovableXY(nFromX,nFromY,nFromX + 2,nFromY - 2);
			break;
		case 5	:
			if ( GetTeamAt(4,nFromY) != GetTeamAt(nFromX,nFromY) )
				TryToAddMovableXY(nFromX,nFromY,nFromX - 1,nFromY - 1);
			if ( GetTeamAt(4,nFromY - 1) == PS_NOTHING )
				TryToAddMovableXY(nFromX,nFromY,nFromX - 2,nFromY - 2);
			break;
		}
		break;
	}
	// ���� �������� �̵����Ѻ���.
	for ( int k = nFromX + 1; k < DF_PANEL_WIDTH; ++ k )
	{
		// ������ ���� ����ο� ������ �� �̻� �� ����.
		if ( GetTeamAt(k,nFromY) == GetTeamAt(nFromX,nFromY) )
			break;
		TryToAddMovableXY(nFromX,nFromY,k,nFromY);
		if ( GetTeamAt(k,nFromY) != PS_NOTHING )
			break;
	}
	// ���� �������� �̵����Ѻ���.
	for ( k = nFromX - 1; k >= 0; -- k )
	{
		// ������ ���� ����ο� ������ �� �̻� �� ����.
		if ( GetTeamAt(k,nFromY) == GetTeamAt(nFromX,nFromY) )
			break;
		TryToAddMovableXY(nFromX,nFromY,k,nFromY);
		if ( GetTeamAt(k,nFromY) != PS_NOTHING )
			break;
	}

	// ���� �̵����� ����.
	for ( k = nFromY - 1; k >= 0; -- k )
	{
		// ������ ���� ����ο� ������ �� �̻� �� ����.
		if ( GetTeamAt(nFromX,k) == GetTeamAt(nFromX,nFromY) )
			break;
		TryToAddMovableXY(nFromX,nFromY,nFromX,k);
		if ( GetTeamAt(nFromX,k) != PS_NOTHING )
			break;
	}
	
	// �Ʒ��� �̵����Ѻ���.
	for ( k = nFromY + 1; k < DF_PANEL_HEIGHT; ++ k )
	{
		// ������ ���� ����ο� ������ �� �̻� �� ����.
		if ( GetTeamAt(nFromX,k) == GetTeamAt(nFromX,nFromY) )
			break;
		TryToAddMovableXY(nFromX,nFromY,nFromX,k);
		if ( GetTeamAt(nFromX,k) != PS_NOTHING )
			break;
	}
}

void CJKStage::GetMovableMa(int nFromX,int nFromY)
{
	// ���� & �������� �̵��� ����.
	if ( IsPassable(nFromX,nFromY - 1) ) 
		TryToAddMovableXY(nFromX,nFromY,nFromX - 1,nFromY - 2);
	// ���� & �������� �̵��� ����.
	if ( IsPassable(nFromX,nFromY - 1) ) 
		TryToAddMovableXY(nFromX,nFromY,nFromX + 1,nFromY - 2);
	// �Ĺ� & �������� �̵��� ����.
	if ( IsPassable(nFromX,nFromY + 1) ) 
		TryToAddMovableXY(nFromX,nFromY,nFromX - 1,nFromY + 2);
	// �Ĺ� & �������� �̵��� ����.
	if ( IsPassable(nFromX,nFromY + 1) ) 
		TryToAddMovableXY(nFromX,nFromY,nFromX + 1,nFromY + 2);
	// ���� & �������� �̵��� ����.
	if ( IsPassable(nFromX - 1,nFromY) ) 
		TryToAddMovableXY(nFromX,nFromY,nFromX - 2,nFromY - 1);
	// ���� & �Ĺ����� �̵��� ����.
	if ( IsPassable(nFromX - 1,nFromY) ) 
		TryToAddMovableXY(nFromX,nFromY,nFromX - 2,nFromY + 1);
	// ���� & �������� �̵��� ����.
	if ( IsPassable(nFromX + 1,nFromY) ) 
		TryToAddMovableXY(nFromX,nFromY,nFromX + 2,nFromY - 1);
	// ���� & �Ĺ����� �̵��� ����.
	if ( IsPassable(nFromX + 1,nFromY) ) 
		TryToAddMovableXY(nFromX,nFromY,nFromX + 2,nFromY + 1);
}

void CJKStage::GetMovableSang(int nFromX,int nFromY)
{
	// ���� & �������� �̵��� ����.
	if ( IsPassable(nFromX,nFromY - 1) && IsPassable(nFromX - 1,nFromY - 2) ) 
		TryToAddMovableXY(nFromX,nFromY,nFromX - 2,nFromY - 3);

	// ���� & �������� �̵��� ����.
	if ( IsPassable(nFromX,nFromY - 1) && IsPassable(nFromX + 1,nFromY - 2) )
		TryToAddMovableXY(nFromX,nFromY,nFromX + 2,nFromY - 3);
	
	// �Ĺ� & �������� �̵��� ����.
	if ( IsPassable(nFromX,nFromY + 1) && IsPassable(nFromX - 1,nFromY + 2) )
		TryToAddMovableXY(nFromX,nFromY,nFromX - 2,nFromY + 3);
	
	// �Ĺ� & �������� �̵��� ����.
	if ( IsPassable(nFromX,nFromY + 1) && IsPassable(nFromX + 1,nFromY + 2) )
		TryToAddMovableXY(nFromX,nFromY,nFromX + 2,nFromY + 3);
	
	// ���� & �������� �̵��� ����.
	if ( IsPassable(nFromX - 1,nFromY) && IsPassable(nFromX - 2,nFromY - 1) )
		TryToAddMovableXY(nFromX,nFromY,nFromX - 3,nFromY - 2);
	
	// ���� & �Ĺ����� �̵��� ����.
	if ( IsPassable(nFromX - 1,nFromY) && IsPassable(nFromX - 2,nFromY + 1) )
		TryToAddMovableXY(nFromX,nFromY,nFromX - 3,nFromY + 2);
	
	// ���� & �������� �̵��� ����.
	if ( IsPassable(nFromX + 1,nFromY) && IsPassable(nFromX + 2,nFromY - 1) )
		TryToAddMovableXY(nFromX,nFromY,nFromX + 3,nFromY - 2);
	
	// ���� & �Ĺ����� �̵��� ����.
	if ( IsPassable(nFromX + 1,nFromY) && IsPassable(nFromX + 2,nFromY + 1) )
		TryToAddMovableXY(nFromX,nFromY,nFromX + 3,nFromY + 2);
}
/////////////////////////////////////////////////
// HAN : Plus score
// CHO : Minus score
int CJKStage::PayOff()
{
	m_nScore = 0;
	for ( int i,j = 0; j < DF_PANEL_HEIGHT; ++ j )
		for ( i = 0; i < DF_PANEL_WIDTH; ++ i )
		{
			switch ( m_enPanel[j][i] )
			{
			case PS_CHO_JANG		: 
				{
					m_nScore -= DF_SCORE_JANG;
					// ���� �� �տ� �η��� ����
					if ( m_bPoBeforeJang )
					{
						if ( m_enPanel[j - 1][i] == PS_CHO_PO ) 
							m_nScore -= 50;
					}
					break;
				}
			case PS_CHO_SA		: m_nScore -= DF_SCORE_SA;break;
			case PS_CHO_JOL		: 
				{
					m_nScore -= DF_SCORE_JOL;
					// ���� ���� ������ ��������� ������.
					if ( m_bJolToCenter )
					{
						if ( m_enPanel[j][i - 1] == PS_CHO_JOL && i > 5 )
							m_nScore -= 1;
						if ( m_enPanel[j][i + 1] == PS_CHO_JOL &&  i <= 5 )
							m_nScore -= 1;
					}
					// ���� ���� ���� ��
					if ( m_bAggressiveJOL )
					{
						if ( j < 7 )
							m_nScore -= 1;
					}
					break;
				}
			case PS_CHO_PO		: 
				{
					// �� 2,�� 1�̸� �� ������ ����.
					// �� 1,�� 2�̸� �� ������ ����.
					// �׷��� ���� ���� ���� �����̴�.
					m_nScore -= DF_SCORE_PO;
					if ( m_bCHAPOBalance && m_nCHOPoCount < m_nCHOChaCount )
						m_nScore -= 2000;
					// ���� ���� ���� ��
					if ( m_bAggressivePO )
					{
						if ( j < 7 )
							m_nScore -= 1;
					}
					break;
				}
			case PS_CHO_CHA		: 
				{
					// �� 2,�� 1�̸� �� ������ ����.
					// �� 1,�� 2�̸� �� ������ ����.
					// �׷��� ���� ���� ���� �����̴�.
					m_nScore -= DF_SCORE_CHA;
					if ( m_bCHAPOBalance && m_nCHOChaCount < m_nCHOPoCount )
						m_nScore -= 2000;
					// ���� ���� ���� ��
					if ( m_bAggressiveCHA )
					{
						if ( j < 7 )
							m_nScore -= 2;
					}
					break;
				}
			case PS_CHO_MA		: 
				{
					m_nScore -= DF_SCORE_MA;
					// ���� ���� ���� ��
					if ( m_bAggressiveMA )
					{
						if ( j < 7 )
							m_nScore -= 1;
					}
					break;
				}
			case PS_CHO_SANG	: 
				{
					m_nScore -= DF_SCORE_SANG;
					// ���� ���� ���� ��
					if ( m_bAggressiveSANG )
					{
						if ( j < 7 )
							m_nScore -= 1;
					}
					break;
				}
			case PS_HAN_JANG	: 
				{
					m_nScore += DF_SCORE_JANG;
					// ���� �� �տ� �η��� ����
					if ( m_bPoBeforeJang )
					{			
						if ( m_enPanel[j + 1][i] == PS_HAN_PO )
							m_nScore += 50;
					}
					break;
				}
			case PS_HAN_SA		: m_nScore += DF_SCORE_SA;break;
			case PS_HAN_JOL	: 
				{
					m_nScore += DF_SCORE_JOL;
					// ���� ���� ������ ��������� ������.
					if ( m_bJolToCenter )
					{
						if ( m_enPanel[j][i - 1] == PS_HAN_JOL && i > 5 )
							m_nScore += 1;
						if ( m_enPanel[j][i + 1] == PS_HAN_JOL && i <= 5 )
							m_nScore += 1;
					}
					// ���� ���� ���� ��
					if ( m_bAggressiveJOL )
					{
						if ( j > 3 )
							m_nScore += 1;
					}
					break;
				}
			case PS_HAN_PO		:
				{
					// �� 2,�� 1�̸� �� ������ ����.
					// �� 1,�� 2�̸� �� ������ ����.
					// �׷��� ���� ���� ���� �����̴�.
					m_nScore += DF_SCORE_PO;
					if ( m_bCHAPOBalance && m_nHANPoCount < m_nHANChaCount )
						m_nScore += 2000;

					// ���� ���� ���� ��
					if ( m_bAggressivePO )
					{				
						if ( j > 3 )
							m_nScore += 1;
					}
					break;
				}
			case PS_HAN_CHA	: 
				{
					// �� 2,�� 1�̸� �� ������ ����.
					// �� 1,�� 2�̸� �� ������ ����.
					// �׷��� ���� ���� ���� �����̴�.
					m_nScore += DF_SCORE_CHA;
					if ( m_bCHAPOBalance && m_nHANChaCount < m_nHANPoCount )
						m_nScore += 2000;
					// ���� ���� ���� ��
					if ( m_bAggressiveCHA )
					{
						if ( j > 3 )
							m_nScore += 2;
					}
					break;
				}
			case PS_HAN_MA		: 
				{
					m_nScore += DF_SCORE_MA;
					// ���� ���� ���� ��
					if ( m_bAggressiveMA )
					{
						if ( j > 3 )
							m_nScore += 1;
					}
					break;
				}
			case PS_HAN_SANG	: 
				{
					m_nScore += DF_SCORE_SANG;
					// ���� ���� ���� ��
					if ( m_bAggressiveSANG )
					{
						if ( j > 3 )
							m_nScore += 1;
					}
					break;
				}
			}
		}
	// �� �̿ܿ��� �� ���� �߰��� ���� �� ���´�.
	if ( m_bOnlyJangCenter )
	{
		if ( m_enPanel[1][4] != PS_NOTHING && m_enPanel[1][4] != PS_HAN_JANG && m_enPanel[1][5] != PS_HAN_SA )
			m_nScore -= 10;
		if ( m_enPanel[8][4] != PS_NOTHING && m_enPanel[8][4] != PS_CHO_JANG && m_enPanel[8][4] != PS_CHO_SA )
			m_nScore += 10;
	}
	return m_nScore;
}

bool CJKStage::SaveStage(const char * filename)
{
	FILE *fp = fopen(filename,"w+b");
	if ( fp == NULL ) return false;
	
	fwrite(m_enPanel,1,sizeof(m_enPanel),fp);
	m_cHistoryStack.SaveHistory(fp);

	fclose(fp);

	return true;
}

bool CJKStage::LoadStage(const char * filename)
{
	FILE *fp = fopen(filename,"r+b");
	if ( fp == NULL ) return false;

	fread(m_enPanel,1,sizeof(m_enPanel),fp);
	m_cHistoryStack.LoadHistory(fp);
	fclose(fp);

	return true;
}

bool CJKStage::MovePiece(int nFromX,int nFromY,int nToX,int nToY)
{
	if ( nFromX < 0 || nFromX >= DF_PANEL_WIDTH ) return false;
	if ( nFromY < 0 || nFromY >= DF_PANEL_HEIGHT ) return false;
	if ( nToX < 0 || nToX >= DF_PANEL_WIDTH ) return false;
	if ( nToY < 0 || nToY >= DF_PANEL_HEIGHT ) return false;

	m_enPanel[nToY][nToX] = m_enPanel[nFromY][nFromX];
	m_enPanel[nFromY][nFromX] = PS_NOTHING;
	
	return true;
}

MOVERESULT CJKStage::HumanMove(int nFromX,int nFromY,int nToX,int nToY)
{
	PIECES enEatenPiece = m_enPanel[nToY][nToX];

	bool bResult = MovePiece(nFromX,nFromY,nToX,nToY);

	if ( bResult )
	{
		// History ����
		m_cHistoryStack.Push(nFromX,nFromY,nToX,nToY,enEatenPiece);
	}

	if ( !bResult )	return MR_UNMOVED;

	if ( enEatenPiece == PS_HAN_JANG ) return MR_HANDEAD;
	if ( enEatenPiece == PS_CHO_JANG ) return MR_CHODEAD;
	
	return MR_MOVED;
}

bool CJKStage::GetNextPieceLoc(int nFromX,int nFromY,CMoveData & cMD)
{
	if ( nFromX < 0 || nFromX >= DF_PANEL_WIDTH ) return false;
	if ( nFromY < 0 || nFromY >= DF_PANEL_HEIGHT ) return false;
	ClearList();
	cMD.Reset();
	/////////////////////////////////////////////////////
	// �� �� ���� ������ �� �ִ� ������ ���Ѵ�.
	switch ( m_enPanel[nFromY][nFromX] )
	{
	case PS_CHO_JANG	: 
	case PS_CHO_SA		: GetMovableSaAndJang(nFromX,nFromY);break;
	case PS_CHO_JOL		: GetMovableJol(nFromX,nFromY);break;
	case PS_CHO_PO		: GetMovablePo(nFromX,nFromY);break;
	case PS_CHO_CHA		: GetMovableCha(nFromX,nFromY);break;
	case PS_CHO_MA		: GetMovableMa(nFromX,nFromY);break;
	case PS_CHO_SANG	: GetMovableSang(nFromX,nFromY);break;
	case PS_HAN_JANG	: 
	case PS_HAN_SA		: GetMovableSaAndJang(nFromX,nFromY);break;
	case PS_HAN_JOL		: GetMovableJol(nFromX,nFromY);break;
	case PS_HAN_PO		: GetMovablePo(nFromX,nFromY);break;
	case PS_HAN_CHA		: GetMovableCha(nFromX,nFromY);break;
	case PS_HAN_MA		: GetMovableMa(nFromX,nFromY);break;
	case PS_HAN_SANG	: GetMovableSang(nFromX,nFromY);break;
	}
	cMD.SetFromXY(nFromX,nFromY);

	if ( m_pFirst )
	{
		CJKStage * pNode = m_pFirst;
		do
		{
			cMD.AddToXY(pNode->m_nToX,pNode->m_nToY);
			pNode = pNode->m_pNext;
		} while ( pNode != NULL );
		return true;
	}
	return true;
}

void CJKStage::GetNextStage()
{
	if ( m_enTeam == TM_HAN )
	{
		for ( int j,i = 0; i < DF_PANEL_HEIGHT; ++ i )
		{
			for ( j = 0; j < DF_PANEL_WIDTH; ++ j )
			{
				switch ( m_enPanel[i][j] )
				{
				case PS_HAN_JANG	: 
				case PS_HAN_SA		: GetMovableSaAndJang(j,i);break;
				case PS_HAN_JOL		: GetMovableJol(j,i);break;
				case PS_HAN_PO		: GetMovablePo(j,i);break;
				case PS_HAN_CHA		: GetMovableCha(j,i);break;
				case PS_HAN_MA		: GetMovableMa(j,i);break;
				case PS_HAN_SANG	: GetMovableSang(j,i);break;
				}
			}
		}
	} else
	{
		for ( int j,i = 0; i < DF_PANEL_HEIGHT; ++ i )
		{
			for ( j = 0; j < DF_PANEL_WIDTH; ++ j )
			{
				switch ( m_enPanel[i][j] )
				{
				case PS_CHO_JANG	: 
				case PS_CHO_SA		: GetMovableSaAndJang(j,i);break;
				case PS_CHO_JOL		: GetMovableJol(j,i);break;
				case PS_CHO_PO		: GetMovablePo(j,i);break;
				case PS_CHO_CHA		: GetMovableCha(j,i);break;
				case PS_CHO_MA		: GetMovableMa(j,i);break;
				case PS_CHO_SANG	: GetMovableSang(j,i);break;
				}
			}
		}
	}
}

bool CJKStage::CutOffTest()
{
	if ( m_enEatenPiece == PS_CHO_JANG ) return true;
	if ( m_enEatenPiece == PS_HAN_JANG ) return true;
	
	////////////////////////////////////////////////////////////
	// FeedOver�� �� �� �ΰ��� �� ���� �����϶� �׸��־� �Ѵ�.
	// ��ǻ�� ���� �����̶� �׸� �� ��� �� ���� �ΰ� ������
	// ġ��Ÿ�� ���� ���� �ִ�.
	if ( m_enEatenPiece != PS_NOTHING )
		if ( m_nDepth <= DF_FEEDOVER_DEPTH ) 
			return false;

	if ( m_nDepth >= DF_CUTOFF_DEPTH ) return true;

/*
	if ( m_enEatenPiece == PS_CHO_JANG ) return true;
	if ( m_enEatenPiece == PS_HAN_JANG ) return true;

	/////////////////////////////////////////////////////
	// �⺻������ �� �� �ձ����� ������ ���ٺ���.
	if ( m_nDepth <= DF_CUTOFF_DEPTH ) return false;

	/////////////////////////////////////////////////////
	// FEED OVER Depth���� ���� �߷��� ������ �׸��д�.
	if ( m_nDepth >= DF_FEEDOVER_DEPTH ) return true;

	/////////////////////////////////////////////////////
	// FEED OVER�� �Ѵ�.�� "�ΰ��� �� ����"�� ������ ��
	// �߷��� �׸��д�.Depth �� Ȧ���̸� �ΰ��� �� ��.
//	if ( m_nDepth % 2 == 1 && m_enEatenPiece == PS_NOTHING )
	if ( m_enEatenPiece == PS_NOTHING )
		return true;
*/	
	return true;
	
	return false;
}

int CJKStage::MaxValue(int nAlpha,int nBeta)
{
	// CUTOFF test
	if ( CutOffTest() ) return m_nScore;
	
	////////////////////////////////////////////////////
	// GetNextStage()���� ������ ���� ������� ȿ��������
	// ��Ʈ(���ĺ�Ÿ ����ġ�⿡)�� �� ����Ʈ�� �����д�.
	GetNextStage();
	if ( m_pFirst )
	{
		CJKStage * pNode = m_pFirst;
		int nMinValue;
		// ���� ������ ��� �׽�Ʈ�غ���.
		do
		{
			nMinValue = pNode->MinValue(nAlpha,nBeta);
			if ( nMinValue > nAlpha )
			{
				nAlpha = nMinValue;
				m_pSelectedChild = pNode;
			}
			////////////////////////////////////////////////////////
			// �θ��� Min�� ���� ������ �����Ϸ� �� ���ε� ��(Max)��
			// ���� ������ ������ Beta���� Ŀ������ �θ��� Min�� 
			// ���ñǿ��� ���Ƴ��Ƿ� �� �̻� ���࿡�� �ǹ̰� ����.
			// ��(Max)�� ���� ū ������ ������ ������ ���ϱ� �����̴�.
			if ( nAlpha >= nBeta ) break;
			pNode = pNode->m_pNext;
		} while ( pNode != NULL );
	} 
	// ���õ� ���� ������ �޸𸮻󿡼� �������� �ʴ´�.
	ClearList(m_pSelectedChild);
	return nAlpha;
}

int CJKStage::MinValue(int nAlpha,int nBeta)
{
	// CUTOFF test
	if ( CutOffTest() ) return m_nScore;

	////////////////////////////////////////////////////
	// GetNextStage()���� ������ ���� ������� ȿ��������
	// ��Ʈ(���ĺ�Ÿ ����ġ�⿡)�� �� ����Ʈ�� �����д�.
	GetNextStage();
	if ( m_pFirst )
	{
		CJKStage * pNode = m_pFirst;
		int nMaxValue;
		do
		{
			nMaxValue = pNode->MaxValue(nAlpha,nBeta);
			if ( nMaxValue < nBeta )
			{
				nBeta = nMaxValue;
				m_pSelectedChild = pNode;
			}
			////////////////////////////////////////////////////////
			// �θ��� Max�� ū ������ �����Ϸ� �� ���ε� ��(Min)��
			// ���� ������ ������ Alpha���� ������ �θ��� Max�� 
			// ���ñǿ��� ���Ƴ��Ƿ� �� �̻� ���࿡�� �ǹ̰� ����.
			// ��(Min)�� ���� ���� ������ ������ ������ ���ϱ� �����̴�.
			if ( nBeta <= nAlpha ) break;
			pNode = pNode->m_pNext;
		} while ( pNode != NULL );
	}
	// ���õ� ���� ������ �޸𸮻󿡼� �������� �ʴ´�.
	ClearList(m_pSelectedChild);
	return nBeta;
}

MOVERESULT CJKStage::Infer(TEAM enTeam)
{
	/////////////////////////////////////////////////////
	// ALPHA - Max�� ������ ���� ���� �ĺ��� ��(�ִ�ġ)
	// BETA	 - Min�� ������ ���� ���� �ĺ��� ��(�ּ�ġ)
	/////////////////////////////////////////////////////
	// �ѳ��� Plus �����̹Ƿ� Max�� �Ǿ�� �Ѵ�.
	m_bAmIMax = (enTeam == TM_HAN);
	m_enTeam = enTeam;

	m_nDepth = 0;
	sm_nCount = 0;
	CJKStage *pSelected = NULL;
	/////////////////////////////////////////////////////
	// ���Ĵ� �ִ��� �۰�,��Ÿ�� �ִ��� ũ�� �ʱ�ȭ�����μ�
	// ���ʿ��� ���� ��������� ������ ���� �� �ְ� �Ѵ�.
	// (���ĺ�Ÿ ����ġ�⸦ ������ �ʰ� �Ѵ�.)
	int nAlpha	= -9999999;
	int nBeta	= +9999999;
	int nScore;
	
	ClearList();
	GetNextStage();
	int nSelectedNo;
	////////////////////////////////////////
	// �ѳ��� ���忡�� �߷��Ѵٸ�...
	if ( m_bAmIMax )
	{
		if ( m_pFirst )
		{
			CJKStage * pNode = m_pFirst;
			static int snCount = 0;
			do
			{
				nScore = pNode->MinValue(nAlpha,nBeta);

				CJKStage *pTmp = pNode;
				do
				{
					TRACE("Way No.%d,Depth - %d : %d,%d to %d,%d\n",snCount ,
													pTmp->m_nDepth,
													pTmp->m_nFromX,
													pTmp->m_nFromY,
													pTmp->m_nToX,
													pTmp->m_nToY);
					pTmp = pTmp->m_pSelectedChild;
				} while ( pTmp != NULL );
				TRACE("Way No.%d Score : %d\n\n",snCount ++,nScore);

				if ( nScore > nAlpha )
				{
					nAlpha = nScore;
					pSelected = pNode;
					nSelectedNo = snCount;
				}
				pNode = pNode->m_pNext;
			} while ( pNode != NULL );
		}
	} else
	{
	////////////////////////////////////////
	// ������ ���忡�� �߷��Ѵٸ�...
		if ( m_pFirst )
		{
			CJKStage * pNode = m_pFirst;
			static int snCount = 0;
			do
			{
				nScore = pNode->MaxValue(nAlpha,nBeta);

				CJKStage *pTmp = pNode;
				do
				{
					TRACE("Way No.%d,Depth - %d : %d,%d to %d,%d\n",snCount ,
													pTmp->m_nDepth,
													pTmp->m_nFromX,
													pTmp->m_nFromY,
													pTmp->m_nToX,
													pTmp->m_nToY);
					pTmp = pTmp->m_pSelectedChild;
				} while ( pTmp != NULL );
				snCount ++;
				TRACE("Way No.%d Score : %d\n\n",snCount,nScore);

				if ( nScore < nBeta )
				{
					nBeta = nScore;
					pSelected = pNode;
					nSelectedNo = snCount;
				}

				pNode = pNode->m_pNext;
			} while ( pNode != NULL );
		}

	}
	if ( pSelected )
	{
		TRACE("Selected No. %d with Score %d\n",-- nSelectedNo,nScore);
		m_nFromX		= pSelected->m_nFromX;
		m_nFromY		= pSelected->m_nFromY;
		m_nToX			= pSelected->m_nToX;
		m_nToY			= pSelected->m_nToY;
		m_enEatenPiece	= m_enPanel[m_nToY][m_nToX];

		MovePiece(m_nFromX,m_nFromY,m_nToX,m_nToY);
		// History ����
		m_cHistoryStack.Push(m_nFromX,m_nFromY,m_nToX,m_nToY,m_enEatenPiece);
	
		m_enEatenPiece = pSelected->m_enEatenPiece;
		
		if ( m_enEatenPiece == PS_HAN_JANG ) return MR_HANDEAD;
		if ( m_enEatenPiece == PS_CHO_JANG ) return MR_CHODEAD;

		return MR_MOVED;
	}

	return MR_UNMOVED;
}

bool CJKStage::GetLastMove(SHistory & LMStore)
{
	return m_cHistoryStack.GetLastMove(LMStore);
}

bool CJKStage::MoveBack()
{
	SHistory sH;
	if ( m_cHistoryStack.Pop(sH) )
	{
		m_enPanel[sH.nFromY][sH.nFromX]	= m_enPanel[sH.nToY][sH.nToX];
		m_enPanel[sH.nToY][sH.nToX]		= sH.enEatenPiece;
		return true;
	} 

	return false;
}




